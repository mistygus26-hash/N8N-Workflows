# 📄 Extraction Devis Signés - 100% Local

## ✅ Statut : Production Ready (ACTIF)

**Workflow ID N8N** : `MXmDVXcHxkHXveOU`  
**Dernière mise à jour** : 12 novembre 2025  
**Email dédié** : compta@aurastackai.fr

---

## 🎯 Objectif

Workflow N8N 100% local pour extraire automatiquement les données structurées des devis signés (PDF) via IA (Ollama) et les stocker dans PostgreSQL.

**Avantages de l'approche locale** :
- ✅ RGPD compliant (données sur VPS)
- ✅ Performances optimales
- ✅ Capacité illimitée
- ✅ Coûts maîtrisés

---

## 🏗️ Architecture

### Pipeline de traitement (8 nodes)

```
Surveiller Dossier → Extraire Texte PDF → Extraire Données (AI) 
                                                    ↓
                                          Email par Défaut
                                                    ↓
                                          Insérer Devis (DB)
                                                    ↓
                                          Préparer Items
                                                    ↓
                                    Insérer Lignes Devis (DB)
                                                    ↓
                                    Déplacer vers Processed
```

### Infrastructure

- **N8N** : Orchestration workflow
- **Ollama** : IA locale (modèle `qwen2.5-coder:3b-instruct`)
- **PostgreSQL** : Stockage données
- **NocoDB** : Interface visuelle DB
- **Docker** : Conteneurisation services

---

## 📊 Données Extraites

### Table `devis_signes` (11 attributs)

| Attribut | Type | Requis | Description |
|----------|------|--------|-------------|
| `quote_number` | String | ✅ | Numéro du devis (ex: DEV2024-001) |
| `from_company` | String | ✅ | Entreprise émettrice |
| `from_email` | String | - | Email émetteur (défaut: compta@aurastackai.fr) |
| `to_company` | String | ✅ | Client destinataire |
| `to_email` | String | - | Email client (défaut: compta@aurastackai.fr) |
| `quote_date` | String | ✅ | Date devis (format YYYY-MM-DD) |
| `total_ht` | Number | - | Montant HT |
| `total_ttc` | Number | ✅ | Montant TTC |
| `notes` | String | - | Notes/conditions particulières |
| `conditions_payment` | String | - | Conditions paiement |
| `file_path` | String | ✅ | Chemin fichier PDF source |

### Table `lignes_devis` (7 attributs)

| Attribut | Type | Description |
|----------|------|-------------|
| `quote_number` | String | Lien avec devis parent |
| `product_name` | String | Nom produit/service |
| `description` | String | Description détaillée |
| `item_type` | String | Type (service/product) |
| `quantity` | Number | Quantité |
| `unit_price` | Number | Prix unitaire |
| `total` | Number | Total ligne |

---

## 📁 Structure Filesystem

```
/opt/devis/
├── uploads/        ← PDFs déposés ici (surveillance active)
└── processed/      ← PDFs traités archivés
```

**Fonctionnement** :
1. Déposer PDF dans `/opt/devis/uploads/`
2. Workflow déclenché automatiquement
3. Extraction + stockage DB
4. PDF déplacé vers `processed/`

---

## 🔧 Configuration Actuelle

### Credentials N8N

**PostgreSQL** (ID: tcC1YiVO1oWf0FY6)
- Host: `n8n-postgres-prod` (Docker network)
- Database: `n8n_db`
- Schema: `public`

**Ollama** (ID: 1xrE2YyJE2Fvcnof)
- URL: `http://ollama:11434` (Docker network)
- Modèle: `qwen2.5-coder:3b-instruct`
- ⚠️ **Limite VPS** : 3B params max (32B models = crash)

### Workflow Settings

- **Timezone** : Europe/Paris
- **Execution Order** : v1
- **Save Executions** : Toutes (succès + erreurs)
- **Active** : ✅ OUI

---

## 🚀 Utilisation

### Test rapide

```bash
# Copier un PDF de test
cp /chemin/vers/devis.pdf /opt/devis/uploads/

# Vérifier les logs N8N
docker logs -f n8n --tail 50

# Vérifier l'insertion DB via NocoDB
# Accès : https://nocodb.aurastackai.fr
```

### Vérification base de données

```bash
# Se connecter à PostgreSQL
docker exec -it n8n-postgres-prod psql -U n8n_user -d n8n_db

# Lister les devis
SELECT quote_number, from_company, to_company, total_ttc 
FROM devis_signes 
ORDER BY created_at DESC 
LIMIT 10;

# Lister les lignes d'un devis
SELECT product_name, quantity, unit_price, total 
FROM lignes_devis 
WHERE quote_number = 'DEV2024-XXX';
```

---

## 📈 Prochaines Évolutions

Voir le fichier [ROADMAP.md](./ROADMAP.md) pour la feuille de route détaillée.

**Priorité Haute** :
1. ✉️ **Intégration email** : Réception automatique des devis via compta@aurastackai.fr
2. 🧾 **Module facturation** : Pipeline similaire pour les factures

---

## 🛠️ Troubleshooting

### Le workflow ne se déclenche pas

```bash
# Vérifier que le workflow est actif
curl -X GET http://localhost:5678/api/v1/workflows/MXmDVXcHxkHXveOU \
  -H "X-N8N-API-KEY: your-api-key"

# Vérifier les permissions du dossier
ls -la /opt/devis/uploads/

# Tester manuellement le workflow dans N8N
# Interface → "Test Workflow" avec un PDF exemple
```

### Erreur Ollama "Model not found"

```bash
# Vérifier les modèles disponibles
docker exec ollama ollama list

# Télécharger le modèle si absent
docker exec ollama ollama pull qwen2.5-coder:3b-instruct
```

### Erreur PostgreSQL "Connection refused"

```bash
# Vérifier que le conteneur est démarré
docker ps | grep postgres

# Tester la connexion réseau
docker exec n8n wget -O- http://n8n-postgres-prod:5432

# Recréer la credential dans N8N si nécessaire
```

---

## 📞 Support

Email technique : admin@aurastackai.fr  
Email métier : compta@aurastackai.fr

---

## 📝 Changelog

### v1.0.0 - 12 novembre 2025
- ✅ Workflow production ready activé
- ✅ 11 attributs métier configurés
- ✅ Email par défaut (compta@aurastackai.fr)
- ✅ Documentation complète
- ✅ Tests de bout en bout validés

### v0.9.0 - 9 novembre 2025
- 🔧 Configuration initiale workflow
- 🔧 Tables PostgreSQL créées
- 🔧 Intégration Ollama 3B
- 🔧 Filesystem monitoring configuré