# 🏗️ Architecture Technique

## Vue d'ensemble

Ce document détaille l'architecture technique du système d'extraction automatique des devis signés.

---

## 📦 Stack Technique

### Infrastructure

```yaml
VPS Debian:
  Docker Containers:
    - N8N (orchestration workflows)
    - PostgreSQL (base de données)
    - Ollama (IA locale)
    - NocoDB (interface visuelle DB)
  
  Filesystem:
    - /opt/devis/uploads/     # PDFs entrants
    - /opt/devis/processed/   # PDFs archivés
```

### Réseau Docker

```yaml
Network: n8n-network
  Services:
    - n8n → http://n8n:5678
    - n8n-postgres-prod → postgresql://n8n-postgres-prod:5432
    - ollama → http://ollama:11434
    - nocodb → http://nocodb:8080
```

**Important** : Communication inter-conteneurs via noms de services (pas localhost).

---

## 🔄 Pipeline de Données

### Flux complet

```
PDF déposé → Local File Trigger → Extract Text (PyPDF2) → 
AI Extraction (Ollama) → Set Default Email → Insert DB (devis) → 
Prepare Items → Insert DB (lignes) → Archive PDF
```

### Détail des étapes

#### 1. **Surveillance Filesystem** (Local File Trigger)
- Type : `n8n-nodes-base.localFileTrigger`
- Path : `/opt/devis/uploads`
- Events : `add` (nouveau fichier)
- Polling : `true`
- Output : `{path: "/opt/devis/uploads/devis.pdf"}`

#### 2. **Extraction Texte** (Code Node - Python)
- Librairie : PyPDF2
- Input : chemin PDF
- Output : 
  ```json
  {
    "pdf_text": "Texte intégral du PDF...",
    "filename": "devis.pdf",
    "file_path": "/opt/devis/uploads/devis.pdf"
  }
  ```

#### 3. **Extraction Données IA** (Information Extractor)
- Node : `@n8n/n8n-nodes-langchain.informationExtractor`
- Input : `pdf_text`
- Model : Ollama Chat Model (qwen2.5-coder:3b-instruct)
- Schema : 11 attributs définis
- Retry logic : 3 tentatives auto
- Output : JSON structuré

#### 4. **Enrichissement Email** (Code Node - JS)
- Logique : Si email vide → compta@aurastackai.fr
- Appliqué sur : `from_email` et `to_email`

#### 5. **Stockage Base Données** (PostgreSQL Nodes)
- Table `devis_signes` : 1 ligne (header devis)
- Table `lignes_devis` : N lignes (items)
- Transaction : Atomique

#### 6. **Archivage** (Code Node - Python)
- Action : `shutil.move()`
- Source : `/opt/devis/uploads/devis.pdf`
- Destination : `/opt/devis/processed/devis.pdf`

---

## 💾 Modèle de Données

### Schéma PostgreSQL

```sql
-- Table principale
CREATE TABLE devis_signes (
  quote_number VARCHAR(100) PRIMARY KEY,
  from_company VARCHAR(255) NOT NULL,
  from_email VARCHAR(255),
  to_company VARCHAR(255) NOT NULL,
  to_email VARCHAR(255),
  quote_date DATE NOT NULL,
  total_ht NUMERIC(10, 2),
  total_ttc NUMERIC(10, 2) NOT NULL,
  notes TEXT,
  conditions_payment VARCHAR(255),
  file_path TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table lignes
CREATE TABLE lignes_devis (
  id SERIAL PRIMARY KEY,
  quote_number VARCHAR(100) REFERENCES devis_signes(quote_number),
  product_name VARCHAR(255),
  description TEXT,
  item_type VARCHAR(50) DEFAULT 'service',
  quantity NUMERIC(10, 2) DEFAULT 1,
  unit_price NUMERIC(10, 2),
  total NUMERIC(10, 2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index pour performances
CREATE INDEX idx_lignes_quote ON lignes_devis(quote_number);
CREATE INDEX idx_devis_date ON devis_signes(quote_date DESC);
```

### Relations

- **1:N** : 1 devis → N lignes
- **Clé étrangère** : `lignes_devis.quote_number → devis_signes.quote_number`
- **Cascade** : Pas configuré (conservation historique)

---

## 🤖 Configuration IA

### Modèle Ollama

```yaml
Modèle: qwen2.5-coder:3b-instruct
Raison: 
  - 3B params = compatible VPS
  - Optimisé code/structure
  - Multilangue (FR/EN)
  - Rapide inference

À éviter:
  - Modèles 32B+ (crash VPS)
  - Modèles vision (inutile pour PDF texte)
```

### Prompt Engineering

Le node Information Extractor génère automatiquement le prompt basé sur le schéma d'attributs avec :
- Descriptions précises des champs
- Exemples de formats attendus
- Flags `required` pour validation

---

## 🔐 Sécurité & Credentials

### Credentials N8N

**PostgreSQL** :
```yaml
ID: tcC1YiVO1oWf0FY6
Type: postgres
Host: n8n-postgres-prod
Port: 5432
Database: n8n_db
User: n8n_user
SSL: disabled (réseau Docker interne)
```

**Ollama** :
```yaml
ID: 1xrE2YyJE2Fvcnof
Type: ollamaApi
Base URL: http://ollama:11434
Auth: none (réseau Docker interne)
```

### Permissions Filesystem

```bash
# Dossiers accessibles par N8N
chown -R 1000:1000 /opt/devis/
chmod 755 /opt/devis/uploads/
chmod 755 /opt/devis/processed/
```

---

## ⚡ Performance

### Métriques typiques

| Étape | Temps moyen | Goulot |
|-------|-------------|--------|
| Trigger detection | < 1s | Polling interval |
| PyPDF2 extraction | 2-5s | Taille PDF |
| Ollama inference | 8-15s | Modèle 3B |
| PostgreSQL insert | < 1s | - |
| File move | < 0.5s | - |
| **TOTAL** | **12-22s** | **IA** |

### Optimisations possibles

1. **Inotify** au lieu de polling (si supporté)
2. **Batch processing** pour volumes élevés
3. **Modèle quantized** (q4_0) pour Ollama
4. **Connection pooling** PostgreSQL

### Limites VPS

- **RAM** : 3B model OK, 7B+ problématique
- **CPU** : Inference lente sans GPU
- **Stockage** : Surveiller `/opt/devis/processed/`

---

## 🔍 Monitoring

### Logs à surveiller

```bash
# N8N
docker logs -f n8n --tail 100

# Ollama
docker logs -f ollama --tail 50

# PostgreSQL
docker logs -f n8n-postgres-prod --tail 50
```

### Métriques clés

- Taux succès workflow : > 95%
- Temps traitement moyen : < 30s
- Utilisation RAM Ollama : < 2GB
- Espace disque `/opt/devis` : < 80%

### Alertes à configurer

- [ ] Échec workflow N8N
- [ ] Modèle Ollama indisponible
- [ ] PostgreSQL connection errors
- [ ] Disque plein

---

## 🔄 Évolutivité

### Scalabilité verticale

- ✅ Facile : Upgrade VPS (plus RAM/CPU)
- ✅ Modèle IA plus gros (7B/13B si RAM++)

### Scalabilité horizontale

- ⚠️ Complexe : N8N pas nativement distribué
- Solution : Queue system (RabbitMQ/Redis) + workers

### Architecture future (si volume x10)

```
Load Balancer → N8N Instances (x3) → RabbitMQ Queue → 
Ollama Workers (x5) → PostgreSQL HA (Primary + Replica)
```

---

## 📚 Références

- [N8N Documentation](https://docs.n8n.io/)
- [Ollama Models](https://ollama.ai/library)
- [PyPDF2 Docs](https://pypdf2.readthedocs.io/)
- [PostgreSQL Best Practices](https://wiki.postgresql.org/wiki/Performance_Optimization)

---

**Dernière mise à jour** : 12 novembre 2025  
**Version document** : 1.0