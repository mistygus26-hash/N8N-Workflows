# 🤖 N8N Workflows - Automatisation IA

Repository des workflows N8N pour l'automatisation métier avec IA locale.

**Infrastructure** : VPS Debian + Docker (N8N, PostgreSQL, Ollama, NocoDB)  
**Philosophie** : 100% local, RGPD compliant, performances optimales

---

## 📂 Workflows Disponibles

### ✅ Production

#### 📄 [Extraction Devis Signés](./extraction-devis-signes/)
**Statut** : ACTIF (v1.0.0)  
**Description** : Extraction automatique des données structurées des devis PDF via IA (Ollama)

**Fonctionnalités** :
- ✅ Surveillance filesystem en temps réel
- ✅ Extraction texte PDF (PyPDF2)
- ✅ Extraction IA avec 11 attributs métier
- ✅ Stockage PostgreSQL (tables `devis_signes` + `lignes_devis`)
- ✅ Email par défaut (compta@aurastackai.fr)
- ✅ Archivage automatique

**Stack** :
- N8N (orchestration)
- Ollama qwen2.5-coder:3b-instruct (IA)
- PostgreSQL (stockage)
- NocoDB (interface)

[📖 Documentation complète →](./extraction-devis-signes/README.md)

---

### 🚧 En Développement

#### ✉️ Routage Email Comptabilité
**Statut** : Planifié (Phase 2)  
**Description** : Réception automatique des devis/factures par email

**Objectif** : Permettre l'envoi direct à compta@aurastackai.fr au lieu de déposer manuellement les PDFs.

[🗺️ Voir roadmap →](./extraction-devis-signes/ROADMAP.md#phase-2--intégration-email-next)

#### 🧾 Extraction Factures
**Statut** : Planifié (Phase 3)  
**Description** : Pipeline d'extraction similaire pour les factures

**Objectif** : Étendre le système aux factures avec tables dédiées (`factures` + `lignes_factures`).

[🗺️ Voir roadmap →](./extraction-devis-signes/ROADMAP.md#phase-3--module-facturation)

---

## 🏗️ Architecture Globale

### Infrastructure VPS

```yaml
VPS Debian (Docker):
  Services:
    - N8N → Orchestration workflows
    - PostgreSQL → Base de données métier
    - Ollama → IA locale (modèles LLM)
    - NocoDB → Interface visuelle DB
  
  Réseau:
    - Docker network: n8n-network
    - Communication: via noms de services
    - Sécurité: réseau isolé
```

### Filesystem

```
/opt/
├── devis/
│   ├── uploads/        # PDFs entrants devis
│   └── processed/      # PDFs traités devis
└── factures/           # (à venir)
    ├── uploads/
    └── processed/
```

### Stack Technique

| Composant | Version | Rôle |
|-----------|---------|------|
| N8N | Latest | Orchestration workflows |
| PostgreSQL | 14+ | Base de données |
| Ollama | Latest | IA locale (LLM) |
| NocoDB | Latest | Interface DB visuelle |
| PyPDF2 | - | Extraction texte PDF |

---

## 🚀 Quick Start

### 1. Importer un workflow

```bash
# Cloner le repo
git clone https://github.com/mistygus26-hash/N8N-Workflows.git

# Accéder à N8N (interface web)
# Importer le fichier workflow.json via "Import from File"
```

### 2. Configurer les credentials

**PostgreSQL** :
- Host : `n8n-postgres-prod`
- Port : `5432`
- Database : `n8n_db`
- User : `n8n_user`

**Ollama** :
- URL : `http://ollama:11434`
- Modèle : `qwen2.5-coder:3b-instruct`

### 3. Activer le workflow

```bash
# Via interface N8N ou API
curl -X PATCH http://localhost:5678/api/v1/workflows/{id}/activate \
  -H "X-N8N-API-KEY: your-key"
```

---

## 📚 Documentation

Chaque workflow possède sa propre documentation dans son dossier :

- **README.md** : Guide d'utilisation et configuration
- **ARCHITECTURE.md** : Détails techniques d'implémentation
- **ROADMAP.md** : Évolutions futures planifiées
- **workflow.json** : Export N8N importable

---

## 🔧 Maintenance

### Vérifier les services

```bash
# Statut containers Docker
docker ps

# Logs N8N
docker logs -f n8n --tail 100

# Logs Ollama
docker logs -f ollama --tail 50

# Logs PostgreSQL
docker logs -f n8n-postgres-prod --tail 50
```

### Monitoring PostgreSQL

```bash
# Se connecter à la DB
docker exec -it n8n-postgres-prod psql -U n8n_user -d n8n_db

# Vérifier les tables
\dt

# Statistiques devis
SELECT COUNT(*) as total_devis, 
       SUM(total_ttc) as ca_total 
FROM devis_signes;
```

### Gestion modèles Ollama

```bash
# Lister modèles disponibles
docker exec ollama ollama list

# Télécharger un nouveau modèle
docker exec ollama ollama pull qwen2.5-coder:7b-instruct

# Supprimer un modèle
docker exec ollama ollama rm qwen2.5:32b
```

---

## 🛡️ Sécurité & RGPD

### Principes

- ✅ **100% local** : Toutes données sur VPS (pas de cloud)
- ✅ **Réseau isolé** : Communication Docker interne uniquement
- ✅ **Pas d'API externe** : IA hébergée localement (Ollama)
- ✅ **Archivage contrôlé** : Conservation fichiers sources
- ✅ **Audit trail** : Toutes exécutions loggées

### Données sensibles

Les workflows traitent des données financières (devis/factures). Mesures de protection :
- Credentials N8N chiffrés
- Accès PostgreSQL restreint
- Backups réguliers DB
- Logs d'accès

---

## 📊 Statistiques

### Workflows

| Métrique | Valeur |
|----------|--------|
| Workflows actifs | 1 |
| Workflows planifiés | 2 |
| Total nodes | 8 (devis) |
| Temps traitement moyen | 12-22s |
| Taux succès | > 95% |

### Infrastructure

| Ressource | Utilisation | Limite |
|-----------|-------------|--------|
| RAM Ollama | ~1.5GB | 3B model max |
| CPU | Variable | Sans GPU |
| Stockage `/opt` | Variable | Surveiller |

---

## 🤝 Contribution

### Workflow existant

1. Fork le repo
2. Créer une branche feature
3. Modifier le workflow
4. Tester localement
5. Pull request avec documentation

### Nouveau workflow

1. Créer dossier `nom-workflow/`
2. Ajouter `workflow.json`
3. Documenter (README + ARCHITECTURE)
4. Tester en production
5. Pull request

**Important** : Aucune modification en production sans validation préalable.

---

## 📞 Support

**Email technique** : admin@aurastackai.fr  
**Email métier** : compta@aurastackai.fr  
**GitHub Issues** : [Créer un ticket](https://github.com/mistygus26-hash/N8N-Workflows/issues)

---

## 📝 Changelog Global

### v1.0.0 - 12 novembre 2025
- ✅ Workflow extraction devis production ready
- ✅ Documentation complète
- ✅ Architecture technique documentée
- ✅ Roadmap évolutions futures

### v0.9.0 - 9 novembre 2025
- 🔧 Setup infrastructure VPS
- 🔧 Configuration N8N/PostgreSQL/Ollama
- 🔧 Tests initiaux

---

## 📄 Licence

Propriétaire - Aurastack AI (aurastackai.fr)  
Usage interne uniquement.

---

**Dernière mise à jour** : 12 novembre 2025  
**Version repository** : 1.0.0