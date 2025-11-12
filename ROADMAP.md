# 🚀 Roadmap - N8N Workflows

Planification des prochaines évolutions du système d'extraction automatisée.

---

## 📧 Phase 2: Intégration Email (Prioritaire)

### Objectif
Permettre au workflow de recevoir directement les devis/factures par email au lieu de surveiller un dossier.

### État Actuel
- ✅ Le workflow surveille `/opt/devis/uploads` 
- ❌ Pas de réception d'emails directe
- ⚠️ Les PDFs doivent être déposés manuellement dans le dossier

### Solutions Proposées

#### Option 1: Workflow de Routage Email (RECOMMANDÉ)
**Avantages:**
- Flexibilité: peut gérer plusieurs types de documents
- Évolutif: facile d'ajouter de nouveaux workflows
- Séparation des responsabilités: routage vs traitement

**Architecture:**
```
Email Entrant (compta@aurastackai.fr)
    ↓
[Workflow Routeur Email]
    ├─→ Détection type document (devis/facture/autre)
    ├─→ Extraction pièce jointe PDF
    ├─→ Sauvegarde temporaire
    └─→ Routage vers workflow approprié
            ├─→ Workflow Devis (existant)
            └─→ Workflow Factures (à créer)
```

**Composants nécessaires:**
- Node Email Trigger (IMAP)
- Node de détection du type de document (analyse sujet/corps)
- Node d'extraction des pièces jointes
- Node de routage conditionnel

**Configuration requise:**
- Accès IMAP à compta@aurastackai.fr
- Credentials email dans N8N
- Règles de routage basées sur:
  - Sujet de l'email (contient "devis" ou "facture")
  - Expéditeur
  - Pièce jointe (nom du fichier)

#### Option 2: Réception Directe
**Avantages:**
- Plus simple à implémenter
- Moins de nodes

**Inconvénients:**
- Moins flexible
- Un workflow par type de document
- Duplication de la logique email

**À éviter si on prévoit plusieurs types de documents.**

### Actions Concrètes

1. **Configuration Email**
   - [ ] Obtenir les credentials IMAP pour compta@aurastackai.fr
   - [ ] Tester la connexion IMAP depuis N8N
   - [ ] Définir le dossier IMAP à surveiller

2. **Création Workflow Routeur**
   - [ ] Node Email Trigger (IMAP)
   - [ ] Node extraction pièces jointes
   - [ ] Node analyse type document
   - [ ] Node Switch pour routage
   - [ ] Connexions vers workflows existants

3. **Tests**
   - [ ] Test avec email de devis
   - [ ] Test avec email de facture
   - [ ] Test avec email sans pièce jointe
   - [ ] Test avec email avec plusieurs pièces jointes

### Critères de Succès
- ✅ Un email avec devis PDF est automatiquement traité
- ✅ Les données sont extraites et stockées en base
- ✅ Le fichier est archivé
- ✅ Le workflow dévis actuel fonctionne toujours

---

## 💰 Phase 3: Workflow Facturation

### Objectif
Créer un workflow similaire au workflow devis pour traiter les factures.

### État Actuel
- ❌ Aucun workflow pour les factures
- ❌ Pas de tables PostgreSQL pour la facturation
- ❌ Pas de dossiers dédiés

### Architecture Prévue

```
Trigger Email/Dossier → Extraire Texte PDF → Extraire Données IA → 
Définir Email Défaut → Insérer Facture → Préparer Items → 
Insérer Lignes Facture → Archiver Fichier
```

### Différences avec le Workflow Devis

**Similitudes (réutilisables):**
- Extraction PDF avec PyPDF2
- Utilisation d'Ollama pour structuration
- Archivage des fichiers
- Email par défaut

**Différences (à adapter):**
- Attributs spécifiques aux factures:
  - `invoice_number` vs `quote_number`
  - `invoice_date` vs `quote_date`
  - `due_date` (nouveau)
  - `payment_status` (nouveau)
  - `payment_method` (nouveau)
- Tables PostgreSQL différentes:
  - `factures` au lieu de `devis_signes`
  - `lignes_factures` au lieu de `lignes_devis`
- Dossiers différents:
  - `/opt/factures/uploads`
  - `/opt/factures/processed`

### Actions Concrètes

1. **Infrastructure**
   - [ ] Créer tables PostgreSQL pour factures
   - [ ] Créer dossiers `/opt/factures/{uploads,processed}`
   - [ ] Définir les permissions appropriées

2. **Workflow**
   - [ ] Dupliquer le workflow devis
   - [ ] Adapter les attributs pour les factures
   - [ ] Changer les références de tables
   - [ ] Modifier les chemins de dossiers
   - [ ] Tester avec une facture exemple

3. **Intégration**
   - [ ] Connecter au workflow routeur email (Phase 2)
   - [ ] Valider le flux complet email → extraction → stockage

### Attributs à Extraire (estimation)

**Facture principale:**
- `invoice_number` (requis)
- `from_company` (requis)
- `from_email`
- `to_company` (requis)
- `to_email`
- `invoice_date` (requis)
- `due_date`
- `total_ht`
- `total_ttc` (requis)
- `total_tva`
- `payment_status` (paid/pending/overdue)
- `payment_method`
- `notes`

**Lignes facture:**
- `invoice_number` (FK)
- `product_name`
- `description`
- `item_type`
- `quantity`
- `unit_price`
- `tva_rate`
- `total`

### Critères de Succès
- ✅ Une facture PDF est automatiquement traitée
- ✅ Les données sont extraites correctement
- ✅ Le stockage en base fonctionne
- ✅ L'archivage est effectué
- ✅ Les workflows devis et factures coexistent

---

## 📊 Phase 4: Reporting et Analyses (Futur)

### Idées d'Évolutions

**Dashboards NocoDB:**
- Vue d'ensemble des devis/factures
- Statistiques financières
- Suivi des paiements
- Alertes sur factures impayées

**Notifications:**
- Email de confirmation après traitement
- Alertes sur erreurs d'extraction
- Rappels pour factures échues

**Exports:**
- Export comptable vers logiciels tiers
- Rapports mensuels automatisés
- Synthèses PDF

**Améliorations IA:**
- Validation croisée des montants
- Détection d'anomalies
- Suggestions de catégorisation

---

## 🛠️ Maintenance et Optimisation

### Améliorations Continues

**Performance:**
- Optimiser les requêtes PostgreSQL
- Mise en cache des résultats Ollama
- Traitement par batch si gros volumes

**Monitoring:**
- Logs structurés
- Métriques de performance
- Alertes sur échecs

**Sécurité:**
- Audit trail des modifications
- Backup automatique de la base
- Chiffrement des données sensibles

---

**Dernière mise à jour**: 2025-11-12
**Prochaine révision prévue**: Après Phase 2 (intégration email)
