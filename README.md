# N8N Workflows - AurastackAI

Workflows N8N de production pour l'automatisation des processus métier.

## 📋 Workflows Actifs

### 📄 Extraction Devis Signés - 100% Local (v1.1)

**État**: ✅ ACTIF EN PRODUCTION

**Workflow ID N8N**: `MXmDVXcHxkHXveOU`

**Dernière mise à jour**: 2025-11-12

#### Architecture

```
Surveiller Dossier → Extraire Texte PDF → Extraire Données IA → 
Définir Email Défaut → Insérer Devis → Préparer Items → 
Insérer Lignes → Archiver Fichier
```

#### Fonctionnalités

- **Trigger**: Local File Trigger sur `/opt/devis/uploads`
- **Extraction**: PyPDF2 pour extraction texte
- **IA**: Ollama avec qwen2.5-coder:3b-instruct
- **Stockage**: PostgreSQL (tables `devis_signes` + `lignes_devis`)
- **Archivage**: Déplacement automatique vers `/opt/devis/processed`
- **Email**: Valeur par défaut `compta@aurastackai.fr`

#### Données Extraites (11 attributs)

- `quote_number` (requis)
- `from_company` (requis)
- `from_email`
- `to_company` (requis)
- `to_email`
- `quote_date` (requis)
- `total_ht`
- `total_ttc` (requis)
- `notes`
- `conditions_payment`
- `items` (array, requis)

## 🚀 Prochaines Évolutions

### Phase 2: Intégration Email

**Objectif**: Faire arriver les emails directement au workflow

**Options à explorer**:

1. **Workflow de Routage Email**
   - Créer un workflow dédié pour gérer les emails entrants
   - Routage vers le workflow d'extraction selon le type de document
   - Permet de gérer plusieurs types de documents (devis, factures, etc.)

2. **Réception Directe**
   - Configurer l'adresse `compta@aurastackai.fr` pour recevoir directement
   - Nécessite configuration du serveur mail

**État actuel**: Le workflow surveille le dossier `/opt/devis/uploads` uniquement. Il ne reçoit pas encore d'emails directement.

### Phase 3: Facturation

**À implémenter**:
- Création d'un workflow similaire pour les factures
- Nouvelles tables PostgreSQL pour les données de facturation
- Dossiers dédiés `/opt/factures/uploads` et `/opt/factures/processed`

**Note**: Actuellement, seule la partie DEVIS est implémentée. La facturation est manquante.

## 🔧 Infrastructure

### Services Docker

- **N8N**: Orchestrateur de workflows
- **PostgreSQL**: Base de données
- **Ollama**: Modèle IA local (qwen2.5-coder:3b-instruct)
- **NocoDB**: Interface visuelle pour PostgreSQL

### Configuration

- **Timezone**: Europe/Paris
- **Exécutions**: Toutes sauvegardées (succès + erreurs)
- **Réseau**: Inter-container via noms Docker
- **Stockage**: 100% local, conforme RGPD

---

**Contact**: compta@aurastackai.fr
