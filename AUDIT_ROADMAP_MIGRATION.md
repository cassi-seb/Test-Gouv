# 📋 AUDIT & ROADMAP MIGRATION APPS POWER BI

**Contexte** : 15 workspaces, 20 rapports, RLS déjà en place, objectif UX  
**Timeline** : 6-8 semaines (1.5 mois)  
**Effort** : 26-44 jours = 3-5.5 ETP semaines

---

## PHASE 0 : AUDIT BASELINE (Semaine 1)
**Objectif** : État actuel complet + décision architecture  
**Effort** : 4-6 jours  
**Outputs** : Audit report + architecture decision

---

## PHASE 1 : DESIGN (Semaine 2)
**Objectif** : Architecture apps + audiences + gouvernance  
**Effort** : 6-11 jours  
**Outputs** : Design doc + RLS validation + policy

---

## PHASE 2 : IMPLÉMENTATION (Semaine 3-4)
**Objectif** : Créer apps, tester, pilot  
**Effort** : 9-17 jours  
**Outputs** : Apps ready, pilot feedback

---

## PHASE 3 : GO-LIVE (Semaine 5)
**Objectif** : Formation + migration + décommissionner  
**Effort** : 7-10 jours  
**Outputs** : Migration complète

---

# 📊 PHASE 0 : AUDIT BASELINE (SEMAINE 1)

## 0.1 EXTRACTION API PBI

### Tâche : Récupérer état actuel complet

```sql
-- TABLE 1 : WORKSPACES (15)
SELECT
  workspace_id,
  workspace_name,
  workspace_type,  -- "métier" vs "équipe"
  owner,
  created_date,
  capacity_type,   -- Premium ou Shared
  is_active
FROM workspaces;

-- TABLE 2 : DATASETS (15-25 estimé)
SELECT
  dataset_id,
  dataset_name,
  workspace_id,
  owner,
  is_refreshing,
  rls_enabled,     -- OUI/NON (tu dis que OUI sur sensibles)
  rls_dimension,   -- "Métier" ou "Fonction" ou autre
  created_date
FROM datasets;

-- TABLE 3 : REPORTS (20)
SELECT
  report_id,
  report_name,
  workspace_id,
  dataset_id,
  owner,
  created_date
FROM reports;

-- TABLE 4 : WORKSPACE ACCESS (droits actuels)
SELECT
  workspace_id,
  principal_id,      -- User email ou Group AD DN
  principal_type,    -- "User" ou "Group"
  access_right,      -- Admin, Member, Contributor, Viewer
  added_date,
  is_individual_user -- Flag pour identifier dérives
FROM workspace_access;

-- TABLE 5 : RLS ROLES
SELECT
  dataset_id,
  role_name,
  dimension_value,   -- Ex: "France", "Finance", "Manager"
  members            -- Populations assignées
FROM rls_roles;
```

**Effort** : 1-2 jours (extraction + nettoyage)

---

## 0.2 AUDIT DÉRIVES & ÉTAT ACTUEL

### Contrôle 1 : Users individuels (shadow IT)

```sql
SELECT
  workspace_id,
  workspace_name,
  principal_id,
  principal_type,
  access_right,
  'INDIVIDUAL USER' as finding
FROM workspace_access
WHERE principal_type = 'User'    -- Pas un groupe AD
ORDER BY workspace_id;

-- Résultat attendu : ~0 (si tu dis que c'est clean)
```

**Question** : Combien de users individuels actuellement ? (0 ou plusieurs ?)

### Contrôle 2 : Admins autorisés

```sql
SELECT
  workspace_id,
  workspace_name,
  principal_id,
  access_right,
  CASE WHEN principal_id NOT IN ('équipe-data@company', 'pbi-admins@company')
       THEN 'UNAUTHORIZED ADMIN'
       ELSE 'OK'
  END as status
FROM workspace_access
WHERE access_right = 'Admin';
```

**Question** : Admins non-équipe data ? (oui/non ?)

### Contrôle 3 : RLS coverage

```sql
SELECT
  COUNT(*) as total_datasets,
  SUM(CASE WHEN rls_enabled = 1 THEN 1 ELSE 0 END) as with_rls,
  SUM(CASE WHEN rls_enabled = 0 THEN 1 ELSE 0 END) as without_rls,
  ROUND(100.0 * SUM(CASE WHEN rls_enabled = 1 THEN 1 ELSE 0 END) / COUNT(*), 1) as rls_percentage
FROM datasets;

-- Résultat attendu : 100% RLS sur sensibles (tu dis déjà en place)
```

**Résultat attendu** : "RLS déjà sur les sensibles ✅"

### Contrôle 4 : Mapping actuel

```
Table simple :
Workspace | Dataset | Reports | RLS? | Populations | Notes
────────────────────────────────────────────────────────────────
Finance  | Budget  | R1, R2  | OUI  | france@, finance@ | OK
Commercial| Sales | R3, R4  | OUI  | commercial@      | OK
RH       | HR Data | R5      | OUI  | rh@              | OK
...
```

**Effort** : 1-2 jours (créer mapping)

---

## 0.3 SYNTHÈSE AUDIT

### Checklist validation

```
✅ 15 workspaces identifiés
   └─ Par métier/équipe ? Distribution OK ?

✅ 20 rapports localisés
   └─ Distribution par workspace ? Avg = 1-3 rapports/WS ?

✅ Datasets complets
   └─ Total : ___ datasets
   └─ RLS coverage : __% (tu dis ~100% sur sensibles)

✅ Droits actuels
   └─ Users individuels : ___ (tu dis ~0)
   └─ Admins non-autorisés : ___ (tu dis ~0)

✅ RLS validée
   └─ RLS dimension : par métier ? par fonction ? mixte ?
   └─ Efficacité : test data leakage = ✅ OK

✅ Populations hiérarchie
   └─ Mères : ___ (France, Commercial, etc.)
   └─ Filles : ___ (Finance, Équipe, etc.)
   └─ Mapping complexité : Simple / Moyen / Complex ?
```

---

## 0.4 DÉCISION ARCHITECTURE

### Option A : 1:1 Mapping (Workspace = App)

```
15 workspaces → 15 apps

Avantages :
├─ Zéro changement structure
├─ Migration rapide (copy-paste)
└─ Risque minimaliste

Inconvénients :
├─ UX pas amélioré (toujours 15 apps = confus)
├─ Gouvernance lourd (15 app owners)
└─ Dénigre l'objectif UX

Recommandation : ❌ PAS bon
```

### Option B : Fusion par Métier (RECOMMANDÉ)

```
15 workspaces → 5-8 apps (par métier/fonction)

Exemple :
├─ App "Finance Reporting" = Finance WS + Finance-Tools WS + datasets
├─ App "Commercial" = Sales WS + Accounts WS
├─ App "RH & Ops" = HR WS + Operations WS
└─ etc.

Avantages :
├─ UX simplifié (5-8 apps claires vs 15 workspaces)
├─ Users voient "Finance" = intuitif
├─ Audiences alignées avec populations
└─ Gouvernance léger (5-8 app owners)

Inconvénients :
├─ Réorganisation workspaces (consolidation)
├─ Mappings complexe si hiérarchie pop compliquée
└─ Effort +5 jours

Recommendation : ✅ MEILLEUR choix
```

### Option C : Hybrid

```
Mélange A + B (certains métiers 1:1, autres fusionnés)

Use case : Si une équipe a besoin workspaces séparées (dev/prod)

Recommandation : ⚠️ À considérer après discussion
```

### DÉCISION À PRENDRE

```
🎯 QUESTION : Quel modèle choisis-tu ?

 A) 1:1 (15 workspaces → 15 apps)
 B) Fusion par métier (15 workspaces → 5-8 apps) ← RECOMMANDÉ
 C) Hybrid (à définir)

📝 Réponse : ________________
```

**Effort** : 1 jour (réunion + décision)

---

## 📊 OUTPUT PHASE 0

### Audit Report (template)

```
RAPPORT AUDIT BASELINE - [DATE]
════════════════════════════════════════════

1. ÉTAT ACTUEL
   ├─ Workspaces : 15
   │  ├─ Métier : __ 
   │  └─ Équipe/Fonction : __
   ├─ Datasets : __ (RLS coverage : __% ✅)
   ├─ Reports : 20
   └─ Populations hiérarchie : __ mères + __ filles

2. DÉRIVES IDENTIFIÉES
   ├─ Users individuels : __ (expected : 0)
   ├─ Admins non-autorisés : __ (expected : 0)
   └─ RLS gaps : NONE ✅ (tu dis déjà en place)

3. ARCHITECTURE DÉCIDÉE
   ├─ Modèle choisi : [A/B/C]
   ├─ Apps résultantes : __ apps
   └─ Justification : [UX, effort, governance]

4. COMPLEXITÉ
   ├─ Hiérarchie populations : Simple/Moyen/Complex
   ├─ Mapping difficulty : Low/Medium/High
   └─ Ressources nécessaires : [évaluation]

5. SIGN-OFF
   ├─ Équipe data : ✅
   ├─ Steering committee : ✅
   └─ Go for Phase 1 : ✅ OUI
```

---

# 🎯 PHASE 1 : DESIGN (SEMAINE 2)

**Objectif** : Définir architecture complète, audiences, gouvernance  
**Effort** : 6-11 jours

---

## 1.1 MAPPING WORKSPACE → APP AUDIENCES

### Tâche 1 : Créer matrice mapping

```
Workspace Name | Type | → APP Name | Audience Groups | Datasets | Reports | Notes
───────────────────────────────────────────────────────────────────────────────────────
Finance WS     | Métier | Finance  | finance@company | Budget, | R1, R2  | Fusion
Finance-Tools  | Équipe |          | france@company  | Actuals | R3      | avec
France-Finance |        |          |                 |         |         | Finance
───────────────────────────────────────────────────────────────────────────────────────
Sales WS       | Métier | Commercial| commercial@    | Sales,  | R4, R5  | Fusion
Accounts WS    |        |          | region@company | Revenue | R6      | ok
───────────────────────────────────────────────────────────────────────────────────────
...
```

**Effort** : 3-5 jours (+ validation métier)

### Tâche 2 : Aligner populations hiérarchiques

```
APP "Finance Reporting"
├─ Audience group = "finance@company.com" (groupe mère)
│  ├─ Members (via AD) : France Finance team, Germany Finance team, etc.
│  └─ Tous les members voient app "Finance"
├─ RLS role = "Finance" (dans dataset Budget)
│  ├─ RLS members : France/Finance sub-pop
│  └─ Voit uniquement ses données (France)
└─ Résultat : Double layer ✅
   ├─ Layer 1 (Audience) : "Es-tu dans finance @ ?"
   └─ Layer 2 (RLS) : "Vois-tu juste ta région ?"

APP "Commercial"
├─ Audience = "commercial@company.com"
│  └─ Members : toutes équipes commercial
├─ RLS = "Commercial" dimension (par région, par client, etc.)
└─ Résultat : Chaque user Commercial voit juste ses données ✅
```

**Effort** : 2-3 jours (validation avec métier)

### Output : Mapping Document

```
TABLE : WORKSPACE TO APP MAPPING

Workspace_ID | Workspace_Name | App_Name | Audience_Group_DN | Datasets | RLS_Status | Owner
─────────────────────────────────────────────────────────────────────────────────────────────
WS001        | Finance        | Finance  | finance@company   | DS001,   | ✅ RLS    | équipe-data
             |                | Report   | france@company    | DS002    | validated |
WS002        | Finance-Tools  | ↑        | ↑                 | ↑        | ↑         |
WS003        | France-FIN     | ↑        | ↑                 | ↑        | ↑         |
─────────────────────────────────────────────────────────────────────────────────────────────
WS004        | Sales          | Commercial| commercial@      | DS003,   | ✅ RLS    | équipe-data
             |                | Dashboards| region@company   | DS004    | validated |
WS005        | Accounts       | ↑        | ↑                 | ↑        | ↑         |
─────────────────────────────────────────────────────────────────────────────────────────────
...
```

---

## 1.2 RLS VALIDATION RAPIDE

### Tâche : Vérifier RLS fonctionne post-migration

```
Pour chaque dataset avec RLS :

✅ Checklist :
  ├─ RLS role dimension clary ?
  ├─ RLS members = populations existantes ?
  ├─ Pas de gap (pop sans RLS role) ?
  ├─ RLS appliquée à tous rapports utilisant dataset ?
  └─ Documentation de RLS dimension ?

Test rapide :
  ├─ Login as User A (France/Finance)
  ├─ Voit France data seulement ? ✅
  ├─ Login as User B (Commercial/Region)
  └─ Voit Region data seulement ? ✅
```

**Effort** : 1-2 jours (c'est juste validation, pas implémentation)

---

## 1.3 GOUVERNANCE POLICY

### Tâche : Définir process

```
GOVERNANCE POLICY
════════════════════════════════════════════

1. APP OWNERS
   ├─ Qui : Équipe data + (optionnel) champion métier
   ├─ Responsabilité :
   │  ├─ Publier app updates
   │  ├─ Gérer audience groups (add/remove populations)
   │  └─ Support users
   └─ Training : 2h workshop avant go-live

2. PUBLICATION PROCESS
   ├─ Checklist avant publication :
   │  ├─ ✅ Audience = groupes AD uniquement (no individuals)
   │  ├─ ✅ RLS appliquée et testée
   │  ├─ ✅ Sensitivity label = "Interne" (min)
   │  └─ ✅ Release notes documentées
   ├─ Approval : Équipe data review (2h max)
   └─ Cadence : Weekly, bi-weekly, ou on-demand ?

3. AJOUT POPULATION / USER
   ├─ Demande : Via IT/DSI ticket
   ├─ Validation : Équipe data + Manager
   ├─ Implémentation : Add user to audience group AD
   ├─ Timeline : 2-3 jours
   └─ No individual access (group only)

4. AUDIT & MONITORING
   ├─ Frequency : Monthly (ou weekly ?)
   ├─ Check : Audience groups coherent ?
   ├─ Alert : Individual user access detected ?
   └─ Escalation : Équipe data

5. DECOMMISSION
   ├─ Old workspaces : Archive, then delete (30-day retention)
   ├─ Users : Migrate to app audience
   └─ Timeline : 30 days grace period
```

**Effort** : 1-2 jours (documentation)

---

## 📊 OUTPUT PHASE 1

```
✅ Design Document (4 pages)
   ├─ Workspace → App mapping (table)
   ├─ Audience groups (list)
   ├─ RLS validation results
   └─ Governance policy

✅ Sign-off
   ├─ Équipe data : ✅
   └─ Steering : ✅ Go for Phase 2
```

---

# 💻 PHASE 2 : IMPLÉMENTATION (SEMAINE 3-4)

**Objectif** : Créer apps, tester, pilot  
**Effort** : 9-17 jours

---

## 2.1 CRÉER APPS

### Timeline

```
Week 3 :
├─ Day 1-2 : Créer apps (Power BI Desktop)
│  ├─ Pour chaque app :
│  │  ├─ Créer workspace (dans PBI Service)
│  │  ├─ Publier datasets
│  │  ├─ Publier reports
│  │  └─ Configure app settings
│  └─ Effort : 4-6 hours per app × 5-8 apps = 20-48 hours
├─ Day 3 : Configurer audience groups
│  ├─ Pour chaque app : Add audience group AD
│  └─ Effort : 30 min per app × 5-8 apps = 3-4 hours
└─ Day 4-5 : Internal testing
   ├─ Équipe data : Vérifier apps, data, RLS
   └─ Effort : 1-2 hours per app

Week 4 :
├─ Day 1-2 : Pilot with small audience
│  ├─ Select 2-3 apps (low-risk)
│  ├─ Add 20-50 test users from audience
│  ├─ Survey : "Is app working ? Any issues ?"
│  ├─ Monitor usage logs
│  └─ Effort : 3-5 hours per app
├─ Day 3-4 : Feedback + adjustments
│  ├─ Fix issues
│  ├─ Optimize performance
│  └─ Update documentation
└─ Day 5 : Publish all apps
   └─ Apps ready for production ✅
```

**Effort** : 9-17 jours

---

## 2.2 RLS TEST (QUICK)

### Tâche : Valider RLS fonctionne dans apps

```
Pour chaque app :
  ├─ Login as Tester from Population A
  ├─ Vérifier : Sees app ? ✅
  ├─ Vérifier : Sees ONLY their data in reports ? ✅
  ├─ Login as Tester from Population B
  ├─ Vérifier : Sees DIFFERENT app ? ✅
  └─ Vérifier : No data cross-leakage ? ✅

Test Matrix :
  User Type | App Access | Data Visibility | RLS Check
  ──────────────────────────────────────────────────────
  Finance   | Finance ✅ | Finance only ✅ | ✅
  Commercial| Comm ✅    | Comm only ✅    | ✅
  RH        | RH ✅      | RH only ✅      | ✅
```

**Effort** : 1-2 hours (since RLS already validated in Phase 1)

---

## 📊 OUTPUT PHASE 2

```
✅ 5-8 Apps created & tested
✅ Audiences configured
✅ RLS validated working
✅ Pilot feedback collected
✅ All ready for production

→ Go for Phase 3 (GO-LIVE)
```

---

# 🚀 PHASE 3 : GO-LIVE (SEMAINE 5)

**Objectif** : Formation + Migration + Décommissionner  
**Effort** : 7-10 jours

---

## 3.1 FORMATION USERS

```
Day 1 :
├─ "Finding Your App" training (30 min, recorded)
│  ├─ "Apps are in Power BI home page"
│  ├─ "Click app name to access"
│  ├─ "You see only your reports"
│  └─ "Old workspaces are being retired"
├─ "How to Get Help" (FAQ)
│  └─ Support email / ticket system
└─ Send to all users

Day 2 :
├─ Office hours for questions (1 hour, optional)
├─ Champions network briefing (30 min)
└─ Email : "Apps are live, start using them"
```

**Effort** : 1-2 jours

---

## 3.2 MIGRATION USERS

```
Week of Go-Live :

Day 1-2 :
├─ Add all users to app audience groups (via AD)
├─ Users automatically see apps in PBI home
└─ Old workspaces still visible (parallel period)

Day 3-4 :
├─ Monitor adoption (users opening apps ?)
├─ Support : Answer questions
└─ Capture any issues

Day 5 :
├─ Review adoption metrics
├─ Fix any issues
└─ Plan decommission
```

**Effort** : 3-5 jours

---

## 3.3 DÉCOMMISSIONNER WORKSPACES

```
Week 2 after Go-Live :

├─ Archive old workspaces (rename → "[RETIRED]")
├─ Remove user access from old workspaces
├─ Keep 30-day retention (for emergency access)
├─ Email : "Old workspaces will be deleted [DATE]"

Week 5 after Go-Live :

├─ Delete old workspaces (after 30-day retention)
├─ Archive audit logs
└─ Confirm all users on apps ✅
```

**Effort** : 2-3 jours

---

## 📊 OUTPUT PHASE 3

```
✅ All users migrated to apps
✅ Adoption ≥ 70% (in first week)
✅ Old workspaces decommissioned
✅ Support tickets handled

→ Migration Complete ✅
```

---

# 📈 SUCCESS METRICS

```
Measure What ?
─────────────────────────────────────────────────────────────
Adoption       | % users opening app / week (Target : ≥70%)
Satisfaction   | NPS score on "Finding app"
Performance    | App load time (Target : <3 sec)
RLS Validation | No data leakage incidents (Target : 0)
Governance     | All apps using group audiences (Target : 100%)
Support        | Tickets resolved in <1 day (Target : 95%)
```

---

# 🎯 CRITICAL SUCCESS FACTORS

```
✅ RLS already in place (no implementation needed)
✅ Clear app architecture (5-8 apps vs 15 workspaces)
✅ Audience groups aligned with populations
✅ App owners trained and motivated
✅ Users informed and supported
✅ Governance policy in place
✅ Monitoring & audit baseline established
```

---

# 📅 TIMELINE OVERVIEW

```
Week 1 : AUDIT BASELINE (4-6 jours)
         └─ État actuel + architecture decision

Week 2 : DESIGN (6-11 jours)
         └─ Mapping + RLS validation + governance

Week 3-4: IMPLÉMENTATION (9-17 jours)
         ├─ Créer apps
         ├─ Configure audiences
         ├─ Test + pilot
         └─ Ready for production

Week 5 : GO-LIVE (7-10 jours)
         ├─ Formation users
         ├─ Migrate to apps
         └─ Décommissionner workspaces

════════════════════════════════════════════
TOTAL : 5-6 semaines = 26-44 jours ⚡⚡⚡
```

---

# ✅ NEXT STEPS

```
1. Review this audit & roadmap
2. Answer Phase 0 questions :
   ├─ Individual users currently ? (count)
   ├─ Unauthorized admins ? (count)
   └─ Architecture preference ? (A/B/C)
3. Confirm : Go for Phase 0 audit ?
4. Schedule Phase 0 kickoff meeting
```

---

**Questions ? Let's go !** 🚀
