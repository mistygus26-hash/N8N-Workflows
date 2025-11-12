# 📝 Changelog

Historique des versions et modifications du système d'extraction automatisée.

---

## [v1.1] - 2025-11-12 - PRODUCTION ACTIVE ✅

### 🎉 État Actuel
Workflow de devis **activé en production** et pleinement opérationnel.

### ✨ Ajouté
- **11 attributs configurés** dans le node Information Extractor
- **Email par défaut** `compta@aurastackai.fr` pour les champs vides
- **Documentation complète** sur GitHub:
  - README.md: Vue d'ensemble et statut
  - ARCHITECTURE.md: Infrastructure technique détaillée
  - ROADMAP.md: Prochaines évolutions (email + facturation)
  - workflows/extraction-devis-signes-v1.1-ACTIVE.json: Workflow production

### 🔧 Configuration
- **Workflow ID**: `MXmDVXcHxkHXveOU`
- **Modèle IA**: qwen2.5-coder:3b-instruct (Ollama)
- **Base de données**: PostgreSQL (tables `devis_signes` + `lignes_devis`)
- **Surveillance**: `/opt/devis/uploads`
- **Archivage**: `/opt/devis/processed`

### 📊 Attributs Extraits
1. `quote_number` (requis)
2. `from_company` (requis)
3. `from_email`
4. `to_company` (requis)
5. `to_email`
6. `quote_date` (requis)
7. `total_ht`
8. `total_ttc` (requis)
9. `notes`
10. `conditions_payment`
11. `items` (array, requis)

### 🐛 Problèmes Résolus
- ✅ Node trigger corrigé (`localFileTrigger` au lieu de langchain)
- ✅ Ollama configuré avec le bon modèle (3b au lieu de 32b)
- ✅ Credentials Ollama avec URL Docker correcte (`http://ollama:11434`)
- ✅ Attributs manuellement reconfigurés dans l'interface N8N

---

## [v1.0] - 2025-11-07 - Déploiement Initial

### ✨ Première Version
- Création du workflow d'extraction de devis
- Pipeline complet: détection → extraction → IA → stockage → archivage
- Infrastructure 100% locale (VPS)

### 🏗️ Infrastructure Déployée
- PostgreSQL avec tables `devis_signes` et `lignes_devis`
- Ollama avec modèle qwen2.5-coder
- NocoDB pour interface visuelle
- Dossiers `/opt/devis/{uploads,processed}`

### 🔄 Workflow Initial (8 nodes)
1. Surveiller Dossier Devis (Local File Trigger)
2. Extraire Texte PDF (PyPDF2)
3. Extraire Données Structurées (Information Extractor)
4. Ollama Chat Model
5. Définir Email par Défaut
6. Insérer Devis (PostgreSQL)
7. Préparer Items
8. Insérer Lignes Devis (PostgreSQL)
9. Déplacer vers Processed

### 🎯 Objectifs Atteints
- ✅ Extraction automatisée fonctionnelle
- ✅ Conformité RGPD (100% local)
- ✅ Performance optimale (modèle 3B)
- ✅ Intégrité des données (transactions PostgreSQL)

---

## 🔮 Prochaines Versions

### [v2.0] - À venir - Intégration Email
**Objectif**: Recevoir les devis directement par email

**Fonctionnalités prévues**:
- Workflow routeur email sur `compta@aurastackai.fr`
- Détection automatique du type de document
- Extraction des pièces jointes PDF
- Routage vers workflow approprié

**État**: 🔴 Non démarré

---

### [v3.0] - À venir - Workflow Facturation
**Objectif**: Traiter les factures comme les devis

**Fonctionnalités prévues**:
- Workflow similaire au workflow devis
- Tables PostgreSQL dédiées
- Attributs spécifiques aux factures
- Dossiers `/opt/factures/*`

**État**: 🔴 Non démarré

---

## 📋 Notes de Version

### Compatibilité
- **N8N**: Version production VPS
- **PostgreSQL**: Compatible toutes versions récentes
- **Ollama**: Modèles ≤ 3B paramètres uniquement
- **Python**: PyPDF2 (extraction PDF)

### Limitations Connues
- Interface N8N: Problème de synchronisation API → UI
- VPS RAM: Maximum 3B paramètres pour Ollama
- Email: Pas encore d'intégration directe

### Recommandations
- ✅ Backups réguliers de PostgreSQL
- ✅ Monitoring des exécutions N8N
- ✅ Vérification des logs Docker
- ✅ Tests réguliers avec vrais PDFs

---

## 🔗 Liens Utiles

**Repository GitHub**:
- https://github.com/mistygus26-hash/N8N-Workflows

**Documentation**:
- [README.md](README.md): Vue d'ensemble
- [ARCHITECTURE.md](ARCHITECTURE.md): Infrastructure technique
- [ROADMAP.md](ROADMAP.md): Évolutions futures

**Workflow Production**:
- Fichier: `workflows/extraction-devis-signes-v1.1-ACTIVE.json`
- ID N8N: `MXmDVXcHxkHXveOU`

---

**Maintenu par**: Christophe Cazenave  
**Contact**: compta@aurastackai.fr  
**Dernière mise à jour**: 2025-11-12
