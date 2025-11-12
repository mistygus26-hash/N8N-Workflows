# 🗺️ Roadmap - Évolutions Futures

## 📋 Vue d'ensemble

Ce document trace les évolutions prévues pour le workflow d'extraction de devis/factures.

**Statut actuel** : v1.0.0 - Extraction devis signés (PRODUCTION)  
**Prochaine version** : v2.0.0 - Intégration emails + Facturation

---

## 🎯 Phase 2 : Intégration Email (NEXT)

### Objectif
Permettre au workflow de recevoir directement les devis/factures par email à l'adresse **compta@aurastackai.fr**.

### Deux approches possibles

#### Option A : Workflow de routage email (RECOMMANDÉE)

**Architecture** :
```
Email Trigger (IMAP/POP3) → Filtrer pièces jointes PDF 
                                      ↓
                            Identifier type (devis/facture)
                                      ↓
                        Déposer dans dossier approprié
                                      ↓
              Workflow existant déclenché automatiquement
```

**Avantages** :
- ✅ Séparation des responsabilités
- ✅ Réutilisation workflow existant
- ✅ Facilité de debug/maintenance
- ✅ Un seul point d'entrée email pour tous types de documents

**Stack technique** :
- Email Trigger node N8N (IMAP)
- Extraction pièces jointes
- Move File node pour dispatch

#### Option B : Intégration directe dans workflow actuel

**Architecture** :
```
Email Trigger → Extraire PDF → [reste du pipeline]
```

**Avantages** :
- ✅ Plus simple (moins de workflows)
- ✅ Latence réduite

**Inconvénients** :
- ❌ Workflow unique pour 2 sources (filesystem + email)
- ❌ Complexité accrue du workflow
- ❌ Difficile d'ajouter d'autres sources plus tard

### 📝 Spécifications techniques

**Configuration email requise** :
- Adresse : compta@aurastackai.fr
- Protocole : IMAP (accès lecture)
- Credentials N8N : À créer
- Polling interval : 60 secondes (configurable)

**Filtres à implémenter** :
- Extensions acceptées : `.pdf`
- Taille max : 10 MB (configurable)
- Patterns sujets : keywords "devis", "facture", "quote", "invoice"

**Actions post-traitement** :
- Marquer email comme lu
- Option : Déplacer vers dossier "Traité" (IMAP)
- Option : Archiver pièce jointe originale

### 🚧 Tâches

- [ ] Configurer boîte email compta@aurastackai.fr
- [ ] Créer credential Email (IMAP) dans N8N
- [ ] Développer workflow routage email
- [ ] Tester avec emails réels
- [ ] Documentation procédure
- [ ] Monitoring alertes emails non traités

**Estimation** : 4-6 heures de développement

---

## 🧾 Phase 3 : Module Facturation

### Objectif
Étendre le système pour traiter également les **factures** avec un pipeline similaire.

### Conception

**Nouvelles tables PostgreSQL** :

```sql
-- Table principale factures
CREATE TABLE factures (
  invoice_number VARCHAR(100) PRIMARY KEY,
  from_company VARCHAR(255) NOT NULL,
  from_email VARCHAR(255),
  to_company VARCHAR(255) NOT NULL,
  to_email VARCHAR(255),
  invoice_date DATE NOT NULL,
  due_date DATE,
  total_ht NUMERIC(10, 2),
  total_ttc NUMERIC(10, 2) NOT NULL,
  notes TEXT,
  payment_status VARCHAR(50) DEFAULT 'pending',
  file_path TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table lignes factures
CREATE TABLE lignes_factures (
  id SERIAL PRIMARY KEY,
  invoice_number VARCHAR(100) REFERENCES factures(invoice_number),
  product_name VARCHAR(255),
  description TEXT,
  item_type VARCHAR(50) DEFAULT 'service',
  quantity NUMERIC(10, 2) DEFAULT 1,
  unit_price NUMERIC(10, 2),
  total NUMERIC(10, 2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Workflow dédié** :
- Nom : "📄 Extraction Factures - 100% Local"
- Pipeline identique au workflow devis
- Dossiers : `/opt/factures/uploads/` et `/opt/factures/processed/`

**Attributs extraits** (12 champs) :
- invoice_number ✅
- from_company ✅
- from_email
- to_company ✅
- to_email
- invoice_date ✅
- due_date
- total_ht
- total_ttc ✅
- notes
- payment_status (default: 'pending')
- items (array) ✅

### 🚧 Tâches

- [ ] Créer tables PostgreSQL (factures + lignes_factures)
- [ ] Créer dossiers filesystem `/opt/factures/`
- [ ] Dupliquer workflow devis → factures
- [ ] Adapter prompts IA pour factures
- [ ] Ajouter champ `payment_status`
- [ ] Tester extraction factures réelles
- [ ] Documentation module facturation
- [ ] Interface NocoDB pour factures

**Estimation** : 6-8 heures de développement

---

## 🔮 Phase 4 : Fonctionnalités Avancées (FUTUR)

### Idées en exploration

1. **Rapprochement Devis ↔ Factures**
   - Lien automatique devis → facture correspondante
   - Détection écarts montants
   - Alertes anomalies

2. **Dashboard Analytics**
   - Vue agrégée CA mensuel
   - Top clients
   - Délais paiement moyens
   - Metabase ou Grafana

3. **OCR amélioré**
   - Si besoin traiter scans/photos (pas juste PDF texte)
   - Tesseract OCR node ou API externe

4. **Notifications**
   - Email confirmation traitement
   - Slack/Discord alertes erreurs
   - Webhooks événements métier

5. **Export comptable**
   - Format CSV pour import logiciel compta
   - API vers ERP externe
   - FEC (Fichier Écritures Comptables)

6. **Multi-devises**
   - Support EUR/USD/autres
   - Conversion automatique
   - Taux de change historiques

---

## 📊 Priorités

| Phase | Priorité | Statut | ETA |
|-------|----------|--------|-----|
| 1. Extraction devis | 🔴 Critique | ✅ DONE | - |
| 2. Intégration email | 🟠 Haute | 📋 NEXT | 1 semaine |
| 3. Module facturation | 🟡 Moyenne | 🔜 TODO | 2 semaines |
| 4. Features avancées | 🔵 Basse | 💭 IDEA | TBD |

---

## 📝 Notes

- **Contrainte VPS** : Modèles IA limités à 3B params max
- **RGPD** : Toutes données restent sur infra locale
- **Évolutivité** : Architecture modulaire pour ajouts futurs
- **Maintenance** : Aucune modification workflow sans validation

---

## 🤝 Contribution

Ce document est évolutif. Toute suggestion d'amélioration est bienvenue :
- Email : admin@aurastackai.fr
- GitHub Issues : [lien repo]

---

**Dernière mise à jour** : 12 novembre 2025  
**Version document** : 1.0