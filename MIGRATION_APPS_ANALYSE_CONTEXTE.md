# 📊 Analyse Migration Apps - Contexte Orga Spécifique

**Contexte** : 15 workspaces, 20 rapports, populations hiérarchiques imbriquées, RLS variable par métier/fonction  
**Objectif principal** : **UX/Ergonomie** → users ne voient que ce qui les concerne (+ pas de tentation)

---

## 1️⃣ STRUCTURE ACTUELLE (WORKSPACE DIRECT)

### 1.1 Composition globale

```
WORKSPACES (15)
├─ 8-10 workspaces métier (Finance, Commercial, RH, Operations, etc.)
└─ 5-7 workspaces équipe/fonction (Équipes spécialisées, transverses)

DATASETS (estimation : 15-25)
├─ Alguns sur tous les workspaces (données partagées)
└─ Certains dédiés à 1-2 workspaces

REPORTS (20 total)
├─ Distribution : 1-3 rapports par workspace en moyenne
└─ Accès : via workspace direct (viewers peuvent lire)

POPULATIONS (hiérarchie imbriquée)
├─ Mères (région, grande fonction)
├─ Filles (équipes, métiers, sous-fonctions)
└─ Mixte (certaines équipes sont imbriquées différemment)

RLS STATUS
├─ Sur certains datasets (pas tous)
├─ Dimensionnée par : métier OU fonction OU combinaison
└─ Efficacité : à valider (fuites possibles ?)
```

### 1.2 Accès actuels (PAIN POINTS UX)

```
PROBLÈME 1 : Workspace Sprawl
├─ Users voient 15 workspaces en interface PBI
├─ Doivent chercher le "bon" workspace
├─ Confusion : "Suis-je dans le bon endroit ?"
└─ Risque : accès involontaire à données non-pertinentes

PROBLÈME 2 : Pas de séparation Dev/Prod
├─ Workspace = à la fois travail ET distribution
├─ Users voient rapports en brouillon
└─ Équipe data doit gérer manuellement les versions

PROBLÈME 3 : RLS incohérente
├─ RLS sur certains datasets, pas tous
├─ Dimensions RLS ≠ hiérarchie populations
├─ Users voir données cross-métier par accident
└─ Exemple : User France/Finance voit aussi Commercial ?

PROBLÈME 4 : Hiérarchie populations ≠ Workspaces
├─ Populations imbriquées complexement
├─ Workspace access = par population mère OU fille ?
├─ Pas de clarté : qui doit avoir accès à quoi ?
└─ Risk : over-sharing ou under-sharing
```

---

## 2️⃣ AVANTAGES MIGRATION VERS APPS

### 2.1 Pour l'UX/Ergonomie (ENJEU PRIO) ✅

| Avantage | Impact |
|----------|--------|
| **Apps simplifiées** | Users voir 5-8 apps claires (vs 15 workspaces confus) |
| **Navigation intuitive** | Apps = "Finance", "Commercial", "RH" (sémantique métier) |
| **Réduction cognitive** | Pas besoin chercher le bon workspace → gagne 10-15 min/semaine |
| **Accès curé** | Apps contiennent uniquement les rapports pertinents |
| **Pas de dérives** | Users ne voient pas les workspaces (pas envie d'explorer) |
| **Onboarding facile** | Nouveau user = lui assigner group AD → visible en app |
| **Audience claires** | App audience = groupe métier (pas d'ambiguïté) |

### 2.2 Pour la Gouvernance (BÉNÉFICE SECONDAIRE) ✅

| Avantage | Impact |
|----------|--------|
| **Single source of truth** | App = 1 audience group (vs multiple workspace + dataset) |
| **Traçabilité claire** | "Qui voit l'app ?" = super simple à auditer |
| **RLS enforcement** | App audience + RLS alignées → pas de bypass |
| **Nettoyage possible** | Anciens workspaces peuvent être archivés |
| **Audit lightweight** | Pas de workspace sprawl à vérifier |

---

## 3️⃣ INCONVÉNIENTS MIGRATION VERS APPS

### 3.1 Effort & Complexité (COÛTS) ⚠️

| Inconvénient | Sévérité | Mitigation |
|---|---|---|
| **Réorganisation workspaces** | 🔴 HAUT | Design apps ≠ structure actuelle |
| **RLS à harmoniser** | 🔴 HAUT | RLS doit couvrir 100% des datasets (pas 70%) |
| **Hiérarchie populations → app audiences** | 🔴 HAUT | Mapping complexe (imbrication) |
| **Équipe data : besoin plus dédié** | 🟠 MOYEN | App owners doivent publier (vs workspace libre) |
| **Testing RLS exhaustif** | 🔴 HAUT | Tester toutes combos pop × dataset × RLS |
| **Changement user workflow** | 🟠 MOYEN | Users habitués aux workspaces |
| **Power users frustration** | 🟠 MOYEN | Pas d'accès direct au dataset/workspace |

### 3.2 Risques Opérationnels 🚨

| Risque | Probabilité | Mitigation |
|---|---|---|
| **RLS cassée en migration** | 🟠 30% | Test matrix avant, dry-run |
| **Audiences oubliées** | 🟠 20% | Checklist vs populations hiérarchie |
| **Data discrepancy** | 🟡 10% | Validation data avant/après |
| **Adoption lente** | 🟠 25% | Training + champions + incentives |
| **Shadow IT (old workspaces réutilisés)** | 🟡 15% | Décommissioner proprement + comms |

---

## 4️⃣ EFFORT RÉEL (POUR TON CONTEXTE)

### 4.1 Travail sur les DROITS (Workspace → App Audience)

```
TÂCHE 1 : Audit actuel (BASELINE)
├─ Extraire via API PBI : 15 WS × (dataset + access + RLS)
├─ Effort : 2-3 jours
└─ Outputs : MCD actuelle (+ dérives identifiées)

TÂCHE 2 : Mapper workspace → app audiences
├─ Décider : "Quel workspace → quelle app ?" (1:1 ou fusion ?)
├─ Mapper : audience groups par app
│  ├─ App "Finance" audience = {france@company.com, finance@company.com, ?}
│  ├─ Vérifier alignement avec hiérarchie populations
│  └─ Valider : chaque pop a accès à ce qu'il faut
├─ Effort : 3-5 jours (+ validation métier)
└─ Risk : Populations imbriquées = mapping complexe

TÂCHE 3 : Corriger droits AVANT migration
├─ Nettoyer : users individuels → groupes AD
├─ Retirer : admins non-autorisés
├─ Ajouter : populations manquantes
├─ Effort : 1-2 jours (si peu de dérives)
└─ Blocage : dérives nombreuses → nettoyer d'abord

TÂCHE 4 : Créer apps + configurer audiences
├─ Pour chaque app (ex: 8 apps si fusion) :
│  ├─ Créer app dans PBI
│  ├─ Ajouter rapports + datasets
│  ├─ Configurer audience groups
│  └─ Test (small audience d'abord)
├─ Effort : 4-6 jours (1 app = 4-6h)
└─ Parallélisable : équipe data peut faire par batch

TOTAL EFFORT DROITS : 10-16 jours (2-3 semaines)
```

### 4.2 Travail sur la RLS (CRITIQUE) 🔴

```
TÂCHE 1 : Audit RLS actuelle
├─ Quels datasets ont RLS ? (20% ? 50% ? 80% ?)
├─ RLS dimension : métier ? fonction ? région ?
├─ Efficacité : test data leakage (user A voit user B data ?)
├─ Effort : 2-3 jours
└─ Output : "RLS Gap Analysis" (quoi couvrir ?)

TÂCHE 2 : Implémenter RLS manquante (SI NEEDED)
├─ Identifier datasets SANS RLS mais avec données sensibles
├─ Ajouter RLS sur ces datasets
│  ├─ Dimensionnalité : par métier ? par fonction ? par combinaison ?
│  └─ Valider avec équipe métier
├─ Effort : 3-7 jours (dépend nb datasets + complexité)
└─ Risk : RLS mal appliquée = data leakage post-migration

TÂCHE 3 : Harmoniser RLS avec populations hiérarchiques
├─ RLS roles ≠ populations ?
│  ├─ Ex: RLS = "France", "Commercial", "Finance"
│  ├─ Mais populations = "France/Finance", "France/Commercial"
│  └─ Mapper : RLS role = quelle combinaison population ?
├─ Effort : 2-4 jours
└─ Risk : User "France/Finance" voit "France" + "Finance" data (correct ?)

TÂCHE 4 : Test RLS exhaustif (AVANT MIGRATION)
├─ Créer test matrix :
│  ├─ User de chaque population → test chaque app
│  └─ Vérifier : voit ses données + pas données autres
├─ Exemple (20 users × 8 apps = 160 tests)
├─ Effort : 3-5 jours
└─ Critique : RLS cassée = bloquer toute migration

TOTAL EFFORT RLS : 10-19 jours (2-4 semaines)
```

### 4.3 Travail ORGANISATION (Réstructuration)

```
TÂCHE 1 : Décider architecture apps
├─ Option A : 1 app = 1 workspace (1:1) → 15 apps (trop ?)
├─ Option B : 1 app = 1 métier → 8 apps (meilleur UX)
│  └─ Fusion : ex "Finance Workspace 1+2+3" → 1 app "Finance Reporting"
├─ Option C : Hybrid (some métier, some équipe)
├─ Effort : 2-3 jours (décision + validation)
└─ Impact : Détermine tout le reste

TÂCHE 2 : Réorganiser workspaces (SI FUSION DÉCIDÉE)
├─ Consolider datasets (ex: 3 workspace Finance → 1 workspace)
├─ Mover rapports
├─ Mettre à jour connections/refreshes
├─ Effort : 5-10 jours
└─ Risk : Data loss, broken connections (à éviter)

TÂCHE 3 : Gouvernance policy
├─ Qui est app owner ? (équipe data ?)
├─ Processus publication (approval nécessaire ?)
├─ Processus ajout population (qui demande ? qui approuve ?)
├─ Effort : 1-2 jours (doc + validation)
└─ Récurrent : maintenance post-migration

TOTAL EFFORT ORGANISATION : 8-15 jours (1-3 semaines)
```

### 4.4 RÉSUMÉ EFFORT TOTAL

```
Phase 1 : AUDIT + DÉCISION
├─ Audit baseline : 2-3j
├─ Décider architecture : 2-3j
└─ Subtotal : 4-6 jours

Phase 2 : DESIGN + NETTOYAGE
├─ Mapping workspace → app : 3-5j
├─ Audit RLS + gaps : 2-3j
├─ Corriger droits : 1-2j
├─ Gouvernance policy : 1-2j
└─ Subtotal : 7-12 jours

Phase 3 : IMPLÉMENTATION
├─ Implémenter RLS manquante (if any) : 3-7j
├─ Test RLS exhaustif : 3-5j
├─ Créer apps + audiences : 4-6j
├─ Réorganiser workspaces (if needed) : 5-10j
└─ Subtotal : 15-28 jours

Phase 4 : MIGRATION + TEST
├─ Pilot (2-3 apps) : 3-4j
├─ Feedback + ajustements : 2-3j
└─ Subtotal : 5-7 jours

Phase 5 : GO-LIVE + SUPPORT
├─ Formation users + champions : 2-3j
├─ Décommissionner old workspaces : 1-2j
├─ Support post-live (1 semaine) : 3-5j
└─ Subtotal : 6-10 jours

════════════════════════════════════════════════
TOTAL : 37-63 jours = 5-8 semaines (1.5-2 mois)
```

---

## 5️⃣ MATRICE DÉCISION : MIGRATION OUI/NON ?

### Questions clés

```
Q1 : Avez-vous BESOIN d'améliorer l'UX/ergonomie des users ?
     → OUI : Migration intéressante ✅
     → NON : Status quo acceptable (mais audit reste recommandé)

Q2 : Avez-vous RLS sur 100% des datasets sensibles ?
     → NON : Avant migration, implémenter RLS d'abord (effort +10j)
     → OUI : Migration peut commencer directement

Q3 : Pouvez-vous consacrer 2 personnes pendant 2 mois ?
     → OUI : Migration faisable
     → NON : Repousser ou réduire scope (pilot seulement)

Q4 : Équipe data motivation = HIGH ?
     → OUI : App ownership sustainable
     → NON : Risk d'abandon post-migration

Q5 : Populations hiérarchiques = simples à mapper ?
     → OUI : Mapping audiences = rapide (3-5j)
     → NON : Mapper complexe (8-10j) + validation métier longue
```

### Recommandation

```
✅ MIGRER SI :
   ├─ UX est priorité pour org
   ├─ RLS couverture ≥ 80%
   ├─ Ressources disponibles (2j/semaine × 8 semaines)
   └─ Hiérarchie pop peut être clarifiée

⏳ REPOUSSER SI :
   ├─ RLS couverture < 50% (implémenter d'abord)
   ├─ Pas de ressources IT/équipe data
   └─ Hiérarchie pop trop complexe à mapper (risque mapping wrong)

✅ AUDIT OBLIGATOIRE (même sans migration) :
   ├─ Baseline actuelle (où en sommes-nous ?)
   ├─ Identifier dérives (users individuels ?)
   ├─ RLS gap analysis (quoi couvrir ?)
   └─ Recommandations gouvernance
```

---

## 6️⃣ RECOMMANDATION FINALE

### Pour TON contexte (15 WS, 20 rapports, UX = priorité)

```
PHASE 0 : AUDIT LÉGER (Obligatoire) - 1 semaine
├─ Extraction API PBI (15 WS = rapide)
├─ Audit RLS coverage (% datasets protégés)
├─ Identification dérives (users individuels ?)
└─ Outputs : "État actuel" + "Recommendations"

PHASE 1 : DÉCISION ARCHITECTURE - 1 semaine
├─ Réunion : "1 app par workspace" vs "fusion par métier"
├─ Impact UX : montrer mock-ups des 2 options
├─ Décision : steering committee
└─ Outputs : Architecture décidée + buy-in

PHASE 2 : DESIGN DÉTAILLÉ - 2-3 semaines
├─ Mapping workspace → app audiences (complexe si pop imbriquées)
├─ RLS gap analysis + plan implémentation
├─ Gouvernance policy + app owner designation
└─ Outputs : Design document + sign-off

PHASE 3 : IMPLEMENTATION - 4-6 semaines
├─ Implémenter RLS manquante (if any)
├─ Créer apps + configurer audiences
├─ Test exhaustif (RLS + données)
├─ Pilot 2-3 apps (feedback users)
└─ Outputs : Apps ready for production

PHASE 4 : GO-LIVE - 1-2 semaines
├─ Formation users
├─ Migration users vers apps
├─ Décommissionner old workspaces
└─ Outputs : Migration terminée

════════════════════════════════════════════════
TOTAL : 9-15 semaines (2-4 mois) = FAISABLE
```

### Budget approximatif (interne only, pas coaching externe)

```
Équipe data (1 person)    : 100-150 jours
Admin/DSI (0.5 person)    : 20-30 jours
Métier validation (0.3p)  : 10-15 jours
────────────────────────────────────────────
TOTAL                     : 130-195 jours = 6-10 ETP semaines

Si équipe data dédié 50% = 3-5 mois elapsed
Si équipe data dédié 25% = 6-10 mois elapsed
```

---

## 7️⃣ PROCHAINES ÉTAPES

### Pour toi

```
✅ DÉCIDER : Migration oui/non ?
   └─ Si OUI → passer à Phase 0 Audit
   └─ Si NON → au minimum faire audit (sans migration)

✅ CLARIFIER structure populations
   └─ Comment sont imbriquées exactement ?
   └─ Quels groupes AD correspondent aux quelles apps ?

✅ VALIDER RLS coverage
   └─ % datasets avec RLS actuellement ?
   └─ Gaps à couvrir avant migration ?

✅ ESTIMATION EFFORT
   └─ Combien de temps équipe data peut consacrer ?
   └─ Coaching externe possible ? (accélère de 20-30%)
```

### Avec moi

```
✅ SI Migration décidée :
   └─ Créer Audit Léger (PHASE 0)
   └─ Créer Design Template (PHASE 1-2)
   └─ Créer RLS Test Matrix
   └─ Créer Gouvernance Policy

✅ SI Non-migration (audit only) :
   └─ Audit Léger + Recommandations
   └─ Nettoyage + Best Practices
   └─ Monitoring/Audit continu
```

---

## 📌 NOTES IMPORTANTES

**Pour ton contexte spécifique :**

1. **15 workspaces = LIGHT pour migration** (vs 50+)
   → Effort réaliste = 2-3 mois avec 1 person dédié

2. **20 rapports = SIMPLE à gérer**
   → Pas de complexité sur volume

3. **Hiérarchie populations imbriquées = PLUS COMPLEXE**
   → Mapping workspace→app audience = point critique
   → Besoin validation métier (2-3 jours)

4. **RLS variable par métier/fonction = À CLARIFIER**
   → Avant migration, définir RLS model global
   → Sinon risque d'incohérence post-migration

5. **UX = VRAIE PRIORITÉ** ✅
   → Migration **justifiée** pour cet enjeu
   → Bénéfice utilisateur direct (navigation simplifiée)
   → ROI : meilleure adoption + moins de support
