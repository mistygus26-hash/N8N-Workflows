# 🏗️ Architecture Technique

Documentation de l'infrastructure et de l'architecture des workflows N8N.

---

## 🐳 Infrastructure VPS

### Conteneurs Docker Actifs

```
┌─────────────────────────────────────────────────┐
│                   VPS Debian                     │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────┐  ┌─────────────┐              │
│  │    N8N      │  │  PostgreSQL │              │
│  │   (prod)    │──│  (n8n-db)   │              │
│  └─────────────┘  └─────────────┘              │
│         │                                        │
│         │         ┌─────────────┐              │
│         └─────────│   Ollama    │              │
│                   │  (qwen2.5)  │              │
│                   └─────────────┘              │
│                          │                       │
│                   ┌─────────────┐              │
│                   │   NocoDB    │              │
│                   │  (visual)   │              │
│                   └─────────────┘              │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Spécifications VPS

- **OS**: Debian
- **Uptime**: 21 jours, 2h09
- **Load**: 0.25 (excellent)
- **RAM**: 42.4 GB disponible / 47 GB total (90% libre)
- **Disk**: 189 GB disponible / 296 GB total (64% libre)
- **Conteneurs**: 10/10 actifs

### Réseau Docker

**Inter-container:**
- Communication via noms de conteneurs
- Pas de localhost, utilisation des hostnames Docker
- Exemple: `http://ollama:11434` au lieu de `http://localhost:11434`

**Ports exposés:**
- N8N: Port configuré pour accès web
- PostgreSQL: Port interne uniquement
- Ollama: API interne uniquement
- NocoDB: Interface web

---

## 🗄️ Base de Données PostgreSQL

### Configuration

- **Host**: `n8n-postgres-prod` (nom du conteneur)
- **Database**: `n8n_db`
- **Schema**: `public`
- **Timezone**: Europe/Paris

### Tables Existantes

#### `devis_signes`
```sql
CREATE TABLE public.devis_signes (
    id SERIAL PRIMARY KEY,
    quote_number VARCHAR(100) UNIQUE NOT NULL,
    from_company VARCHAR(255) NOT NULL,
    from_email VARCHAR(255),
    to_company VARCHAR(255) NOT NULL,
    to_email VARCHAR(255),
    quote_date DATE NOT NULL,
    total_ht DECIMAL(10,2),
    total_ttc DECIMAL(10,2) NOT NULL,
    notes TEXT,
    conditions_payment VARCHAR(255),
    file_path VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### `lignes_devis`
```sql
CREATE TABLE public.lignes_devis (
    id SERIAL PRIMARY KEY,
    quote_number VARCHAR(100) REFERENCES devis_signes(quote_number),
    product_name VARCHAR(255),
    description TEXT,
    item_type VARCHAR(50) DEFAULT 'service',
    quantity DECIMAL(10,2) DEFAULT 1,
    unit_price DECIMAL(10,2) DEFAULT 0,
    total DECIMAL(10,2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Index et Optimisations

- Index unique sur `quote_number` (PK et FK)
- Index sur `created_at` pour les requêtes chronologiques
- Relation 1-N entre devis et lignes de devis

---

## 🤖 Modèle IA - Ollama

### Configuration

- **Modèle actif**: `qwen2.5-coder:3b-instruct`
- **Endpoint**: `http://ollama:11434`
- **API**: Compatible OpenAI

### Limitations VPS

⚠️ **IMPORTANT**: Le VPS ne peut pas exécuter de modèles > 3B paramètres
- ✅ Modèles 3B: Fonctionnent bien
- ❌ Modèles 7B+: Trop gourmands en RAM
- ❌ Modèles 32B: Impossible à charger

### Performance

- Extraction d'un devis: ~5-10 secondes
- Précision: Bonne sur documents structurés
- Retry automatique: Intégré dans le node Information Extractor

---

## 📁 Système de Fichiers

### Structure des Dossiers

```
/opt/
├── devis/
│   ├── uploads/          # Dépôt des nouveaux PDFs
│   └── processed/        # Archive des PDFs traités
│
└── factures/             # À créer (Phase 3)
    ├── uploads/
    └── processed/
```

### Permissions

- Propriétaire: Utilisateur N8N Docker
- Lecture/Écriture: N8N uniquement
- Surveillance: Local File Trigger (polling)

---

## 🔄 Workflow Actuel: Extraction Devis

### Architecture Détaillée

```
┌─────────────────────────────────────────────────────────────┐
│                    WORKFLOW PRODUCTION                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. [Local File Trigger]                                    │
│     └─ Surveille: /opt/devis/uploads                       │
│     └─ Événements: add (nouveau fichier)                   │
│     └─ Polling: true                                        │
│           │                                                  │
│           v                                                  │
│  2. [Extraire Texte PDF] (Python/PyPDF2)                   │
│     └─ Lit le PDF                                           │
│     └─ Extrait tout le texte                                │
│     └─ Ajoute filename + file_path                         │
│           │                                                  │
│           v                                                  │
│  3. [Extraire Données Structurées] (IA)                    │
│     └─ Ollama qwen2.5-coder:3b-instruct                    │
│     └─ Extraction 11 attributs                              │
│     └─ Validation + Retry automatique                      │
│           │                                                  │
│           v                                                  │
│  4. [Définir Email par Défaut] (Code JS)                   │
│     └─ Si from_email vide → compta@aurastackai.fr         │
│     └─ Si to_email vide → compta@aurastackai.fr           │
│           │                                                  │
│           v                                                  │
│  5. [Insérer Devis] (PostgreSQL)                           │
│     └─ Table: devis_signes                                  │
│     └─ Insert avec mapping des 11 champs                   │
│           │                                                  │
│           v                                                  │
│  6. [Préparer Items] (Code JS)                             │
│     └─ Parse le tableau items[]                             │
│     └─ Crée un objet par ligne de devis                    │
│           │                                                  │
│           v                                                  │
│  7. [Insérer Lignes Devis] (PostgreSQL)                    │
│     └─ Table: lignes_devis                                  │
│     └─ Insert batch de toutes les lignes                   │
│           │                                                  │
│           v                                                  │
│  8. [Déplacer vers Processed] (Python)                     │
│     └─ Source: /opt/devis/uploads/file.pdf                 │
│     └─ Destination: /opt/devis/processed/file.pdf          │
│     └─ Opération: move (pas de copie)                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Attributs Extraits

| Attribut | Type | Requis | Description |
|----------|------|--------|-------------|
| `quote_number` | String | ✅ | Numéro du devis |
| `from_company` | String | ✅ | Entreprise émettrice |
| `from_email` | String | ❌ | Email émetteur |
| `to_company` | String | ✅ | Client destinataire |
| `to_email` | String | ❌ | Email client |
| `quote_date` | Date | ✅ | Date du devis (YYYY-MM-DD) |
| `total_ht` | Number | ❌ | Montant HT |
| `total_ttc` | Number | ✅ | Montant TTC |
| `notes` | String | ❌ | Notes diverses |
| `conditions_payment` | String | ❌ | Conditions de paiement |
| `items` | Array | ✅ | Lignes du devis |

### Gestion des Erreurs

- **Node Information Extractor**: Retry automatique
- **PostgreSQL**: Transactions pour intégrité
- **Archivage**: Gestion des erreurs avec logs
- **Exécutions**: Toutes sauvegardées (succès + erreurs)

---

## 🔧 Configuration N8N

### Settings Workflow

```json
{
  "saveExecutionProgress": true,
  "saveManualExecutions": true,
  "saveDataErrorExecution": "all",
  "saveDataSuccessExecution": "all",
  "timezone": "Europe/Paris",
  "executionOrder": "v1"
}
```

### Credentials

**Ollama API:**
- URL: `http://ollama:11434`
- Pas d'authentification

**PostgreSQL:**
- Host: `n8n-postgres-prod`
- Database: `n8n_db`
- Credentials stockés dans N8N

---

## 🔒 Sécurité et Conformité

### RGPD

✅ **100% Local:**
- Aucune donnée ne quitte le VPS
- Pas d'API externe (sauf Ollama local)
- Pas de cloud tiers (Google Drive, Sheets, etc.)

### Données Sensibles

- Emails chiffrés dans PostgreSQL
- Credentials N8N protégés
- Accès VPS sécurisé
- Backups réguliers recommandés

---

## 📊 Monitoring et Logs

### N8N Exécutions

- **Toutes les exécutions sauvegardées**
- Accès via interface N8N
- Filtres: succès/erreur/en cours
- Historique complet des données

### Logs Docker

```bash
# Voir les logs N8N
docker logs n8n-prod

# Voir les logs Ollama
docker logs ollama

# Voir les logs PostgreSQL
docker logs n8n-postgres-prod
```

### Métriques VPS

- Load average: À surveiller (< 1.0 = bon)
- RAM: 42 GB disponibles
- Disk: 189 GB disponibles
- Conteneurs: Tous up

---

## 🚨 Problèmes Connus

### Interface N8N - Synchronisation

**Symptôme:**
- Les mises à jour via API ne se reflètent pas dans l'interface web
- Les attributs configurés via API apparaissent vides dans l'UI

**Cause:**
- Cache ou problème de synchronisation N8N

**Solution:**
- Reconfigurer manuellement les attributs dans l'interface
- Hard refresh (Ctrl+Shift+R) parfois nécessaire

### Modèles Ollama

**Limitation:**
- VPS limité aux modèles ≤ 3B paramètres
- Ne pas tenter de charger des modèles 7B/13B/32B

---

## 📚 Documentation Technique

### Node Types Utilisés

- `n8n-nodes-base.localFileTrigger`: Trigger sur système de fichiers
- `n8n-nodes-base.code`: Exécution Python/JavaScript
- `@n8n/n8n-nodes-langchain.informationExtractor`: Extraction IA
- `@n8n/n8n-nodes-langchain.lmChatOllama`: Modèle Ollama
- `n8n-nodes-base.postgres`: Opérations PostgreSQL

### Versions

- **N8N**: Version production (VPS)
- **PostgreSQL**: Version conteneur Docker
- **Ollama**: Dernière version compatible
- **PyPDF2**: Installé dans N8N

---

**Dernière mise à jour**: 2025-11-12  
**Workflow ID**: `MXmDVXcHxkHXveOU`  
**Contact**: compta@aurastackai.fr
