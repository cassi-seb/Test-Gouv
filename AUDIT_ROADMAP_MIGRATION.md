# 📋 AUDIT & ROADMAP MIGRATION VERS APPS

**Contexte** : 15 workspaces, 20 rapports, populations hiérarchiques, RLS déjà en place  
**Timeline** : 5-6 semaines (1.5 mois), 26-44 jours effort  
**Objectif** : UX/Ergonomie → users ne voient que ce qui les concerne

---

## RÉSUMÉ EXÉCUTIF

```
✅ RLS déjà en place → pas d'implémentation RLS
✅ 15 workspaces (léger) → migration rapide
✅ 20 rapports (simple) → pas de complexité volume
✅ Effort raisonnable : 26-44 jours (6 semaines)
✅ Risque faible : RLS sécurise déjà les données

RECOMMANDATION : GO pour la migration 🚀
```

---

# PHASE 0 : AUDIT BASELINE (Semaine 1)

**Durée** : 4-6 jours  
**Objectif** : État actuel des droits + décider architecture apps  
**Livrables** : Baseline document + architecture decision

---

## TÂCHE 0.1 : Audit Dérives Actuelles

**Durée** : 2-3 jours  
**Comment** : Manuel (fichier Excel + vérif AD)  
**À faire** :

```
✅ DÉRIVES À IDENTIFIER :

   1. Users INDIVIDUELS en workspace (pas groupe AD)
      └─ Risque : pas de maintenance RH, accès orpheline
      └─ Question : Combien ? (chercher dans Excel, colonne "Access Type")
      └─ Action : Lister tous les users individuels

   2. ADMINS non-équipe data
      └─ Risque : droits excessifs, modifications non-contrôlées
      └─ Question : Y en a combien ? Qui sont-ils ?
      └─ Action : Lister tous les admins non-équipe-data

   3. Populations INACTIVES en RH mais encore en PBI
      └─ Risque : RGPD (données accessibles sans justif)
      └─ Question : Des pops supprimées en RH mais toujours en workspace ?
      └─ Action : Croiser population actives (RH) vs population en PBI

   4. RLS INCOHÉRENTE
      └─ Risque : user voit données cross-métier
      └─ Question : RLS dimension ≠ hiérarchie populations ?
      └─ Action : Vérifier RLS roles alignés avec populations
```

**Output** :
```
Rapport Dérives :
├─ Users individuels : X trouvés
├─ Admins non-autorisés : Y trouvés
├─ Populations inactives : Z trouvés
├─ RLS incohérences : N trouvés
└─ Action plan : nettoyage avant migration (priorité ?)
```

---

## TÂCHE 0.2 : Vérifier RLS Coverage

**Durée** : 1 jour  
**Comment** : Manual (fichier Excel RLS)  
**À faire** :

```
✅ RLS VALIDATION :

   1. Quels datasets ont RLS ? (%)  
      └─ Question : sur 15-25 datasets, combien avec RLS ?
      └─ Réponse attendue : "80%", "100%", "50%" ?

   2. RLS dimension : cohérente avec populations ?
      └─ Exemple : RLS = "France", "Germany" vs populations = "France/Finance", "France/Commercial"
      └─ Question : Mapping correct ?

   3. Datasets SANS RLS mais sensibles ? (budget, perso, etc.)
      └─ Question : Besoin de rajouter RLS après audit ?
      └─ Réponse : "Non" (tu as dit déjà en place)
```

**Output** :
```
RLS Coverage Report :
├─ % datasets avec RLS : X%
├─ RLS dimensions principales : [liste]
├─ Datasets sans RLS : [liste + risque]
└─ Conclusion : "RLS OK pour migration" ✅
```

---

## TÂCHE 0.3 : Décider Architecture Apps

**Durée** : 2-3 jours  
**Comment** : Réunion + analyse  
**À faire** :

```
✅ 3 OPTIONS :

   OPTION A : 1 app = 1 workspace (1:1)
   ├─ Résultat : 15 apps
   ├─ Avantage : Zéro réorganisation
   ├─ Inconvénient : Pas d'amélioration UX (15 apps = toujours confus)
   └─ Recommandation : ❌ NON (défait le but)

   OPTION B : 1 app = 1 métier (fusion)
   ├─ Exemple : 3 "Finance workspaces" → 1 app "Finance Reporting"
   ├─ Résultat : 5-8 apps
   ├─ Avantage : Navigation ultra-claire, UX excellent
   ├─ Inconvénient : Consolider datasets + reconnect (effort +5-10j)
   └─ Recommandation : ✅ OUI (meilleur UX)

   OPTION C : Hybrid
   ├─ Certains métiers = 1 app, autres gardent structure
   ├─ Exemple : Finance fusionne, Commercial reste 1:1
   ├─ Résultat : 8-12 apps
   ├─ Avantage : Compromis
   ├─ Inconvénient : Complexité, inconsistant
   └─ Recommandation : 🟠 PEUT-ÊTRE (dépend contraintes)
```

**Process de décision** :
```
1. Lister les 15 workspaces actuels avec leur métier
   └─ Ex: WS_Finance_1, WS_Finance_2, WS_Commercial, WS_HR, etc.

2. Grouper par métier
   └─ Finance : WS_Finance_1, WS_Finance_2 (2 WS)
   └─ Commercial : WS_Commercial, WS_Sales (2 WS)
   └─ HR : WS_HR (1 WS)
   └─ Etc.

3. Décider : fusionner ou pas ?
   ├─ Si datasets Finance sont indépendants → fusionner en 1 app
   ├─ Si datasets Commercial partagent même audience → fusionner en 1 app
   └─ Si workspace isolé → rester 1:1

4. Valider impact RLS
   └─ Fusion Finance WS1+WS2 → même RLS role ?
   └─ Si oui → fusionner ✅
   └─ Si non → risque RLS cassée ❌
```

**Output** :
```
Architecture Decision :
├─ Option choisie : [A/B/C]
├─ Apps finales : X apps
├─ Mapping workspace → app :
│  ├─ App "Finance" ← WS_Finance_1, WS_Finance_2
│  ├─ App "Commercial" ← WS_Commercial, WS_Sales
│  └─ ...
├─ Rationale : [pourquoi]
└─ Sign-off : [qui valide]
```

---

## TÂCHE 0.4 : Nettoyage Préalable (si dérives trouvées)

**Durée** : 1-2 jours (si peu de dérives)  
**À faire** :

```
✅ NETTOYAGE :

   1. Users individuels → ajouter au groupe AD
      └─ "User Jean" dans workspace → ajouter à groupe "finance@company.com"
      └─ Retirer accès individuel

   2. Admins non-autorisés → retirer ou limiter
      └─ Exemple : User non-équipe-data en Admin → passer en Member

   3. Populations inactives → retirer du workspace
      └─ Exemple : Pop supprimée en RH → retirer du workspace PBI

   4. RLS cassée → corriger (si trouvée)
      └─ Exemple : User voit données cross-métier → valider RLS role
```

**Validation post-nettoyage** :
```
Vérifier :
✅ Tous les users = groupes AD (pas individuels)
✅ Admins = uniquement équipe data
✅ Populations actives en RH = actives en PBI
✅ RLS fonctionne (test user A voit A data, pas B data)
```

---

# PHASE 1 : DESIGN (Semaine 2)

**Durée** : 6-11 jours  
**Objectif** : Design détaillé + gouvernance  
**Livrables** : Design doc + audiences mapping + policy

---

## TÂCHE 1.1 : Mapper Workspaces → App Audiences

**Durée** : 3-5 jours  
**À faire** :

```
✅ POUR CHAQUE APP :

   1. Définir l'audience (groupes AD)
      └─ Exemple App "Finance" :
         ├─ Audience principale : "finance@company.com"
         ├─ Audience secondaire : "france@company.com" (si région applicable)
         ├─ Audience admin : "equipe-data@company.com"
         └─ Valider : chaque groupe a accès au bon app

   2. Valider alignement hiérarchie populations
      └─ Population "France/Finance" → visible en app "Finance" ? ✅
      └─ Population "Commercial/France" → PAS visible en app "Finance" ? ✅
      └─ Pas de cross-leakage ?

   3. Tester audiences manuellement
      └─ Créer user test dans chaque groupe
      └─ Vérifier : "Voit-il juste son app ?"
```

**Output** :
```
Audiences Mapping :
├─ App "Finance" audiences = {finance@company.com, equipe-data@company.com}
├─ App "Commercial" audiences = {commercial@company.com, equipe-data@company.com}
├─ App "HR" audiences = {hr@company.com, equipe-data@company.com}
└─ ... (pour chaque app)

Validation :
✅ Chaque population → bonne app
✅ Pas de cross-access
✅ Admins accessibles partout
```

---

## TÂCHE 1.2 : RLS Validation (Quick Check)

**Durée** : 1-2 jours  
**À faire** :

```
✅ VALIDER RLS ACTUELLE :

   1. RLS = conservée lors migration ?
      └─ Question : Datasets avec RLS restent avec RLS ?
      └─ Réponse : OUI ✅ (pas toucher à RLS)

   2. RLS roles = alignées avec app audiences ?
      └─ Exemple : RLS "France" + audience "france@company.com" = match ?
      └─ Si non → ajouter mapping note

   3. Test data leakage (sample)
      └─ Créer user test "France/Finance"
      └─ Vérifier : voit-il JUSTE France+Finance data ?
      └─ Test 2-3 users représentatifs
```

**Output** :
```
RLS Validation Report :
├─ RLS Status : "OK, conservée, pas de changement"
├─ RLS-App alignment : [mapping]
├─ Data leakage test : "PASS" ✅
└─ Recommendation : "RLS ready for migration"
```

---

## TÂCHE 1.3 : Gouvernance Policy

**Durée** : 1-2 jours  
**À faire** :

```
✅ DÉFINIR PROCESS :

   1. App Owner : Qui ?
      └─ Équipe data uniquement ?
      └─ Ou équipe data + métier designé ?
      └─ Décision : ______

   2. Publication : Approval nécessaire ?
      └─ Équipe data approuve tout ?
      └─ Auto-publish sans approval ?
      └─ Décision : ______

   3. Ajouter population en app : qui demande ?
      └─ Demandeur : User ? Métier ? DSI ?
      └─ Approbateur : Équipe data ? Métier ?
      └─ Process : email ? ticket IT ? form ?
      └─ Décision : ______

   4. Checklist publication app
      └─ ✅ Audience = groupes AD uniquement (pas users individuels)
      └─ ✅ RLS appliquée correctement (si applicable)
      └─ ✅ Sensitivity labels (si besoin)
      └─ ✅ Release notes documentées
      └─ ✅ Équipe data review + sign-off
```

**Output** :
```
Governance Policy v1 :
├─ App Owners : [liste]
├─ Publication Process : [steps]
├─ Access Request Process : [steps]
├─ Checklist : [items]
└─ Sign-off : [date, who]
```

---

## TÂCHE 1.4 : Communication Plan

**Durée** : 1 jour  
**À faire** :

```
✅ COMMUNIQUER CHANGEMENT :

   Semaine 2 (NOW) :
   └─ Email : "Change coming : Workspace → Apps migration"
      ├─ Pourquoi : "Meilleure navigation, UX simplifiée"
      ├─ Quand : "Go-live Semaine 5"
      ├─ Impact : "Workspaces disparaissent, apps arrivent"
      └─ FAQ : "Vais-je perdre mes données ?" → Non, RLS conservée ✅

   Semaine 4 :
   └─ Video : "Comment utiliser les apps" (30 min)
      ├─ Où voir les apps ?
      ├─ Comment chercher mes rapports ?
      ├─ Que se passe-t-il avec mes bookmarks ?
      └─ Support : qui appeler ?

   Semaine 5 :
   └─ Go-live announcement
      ├─ "Apps live, workspaces archived"
      ├─ "Migration réussie ✅"
      └─ "Support disponible pendant 2 semaines"
```

---

# PHASE 2 : IMPLÉMENTATION (Semaine 3-4)

**Durée** : 9-17 jours  
**Objectif** : Créer apps + configurer + tester  
**Livrables** : Apps en prod + test results

---

## TÂCHE 2.1 : Créer Apps & Configurer Audiences

**Durée** : 4-6 jours  
**À faire** :

```
✅ POUR CHAQUE APP (5-8 apps) :

   1. Créer app dans Power BI
      └─ Nom : sémantique métier ("Finance Reporting")
      └─ Workspace : créer ou utiliser existant ?
      └─ Owner : équipe data

   2. Ajouter rapports
      └─ Copier/link rapports depuis ancien workspace
      └─ Tester : rapports affichent-ils ?

   3. Ajouter datasets
      └─ Vérifier datasets accessibles
      └─ RLS check : présente ?

   4. Configurer audience
      └─ Ajouter groupes AD définis en Phase 1
      └─ Test : groupe peut-elle voir l'app ?

   5. Ajouter sensitivity labels (si besoin)
      └─ Exemple : "Finance Sensitive"

   6. Publier (draft mode d'abord)
      └─ Validation équipe avant public
```

**Output** :
```
Apps Created :
├─ App "Finance Reporting" : audiences {finance@company.com, equipe-data@company.com}
├─ App "Commercial Ops" : audiences {commercial@company.com, equipe-data@company.com}
├─ ...
└─ Status : All published (draft) for testing
```

---

## TÂCHE 2.2 : Testing (RLS + Données)

**Durée** : 3-5 jours  
**À faire** :

```
✅ TEST MATRIX (pour 2-3 apps pilot) :

   Créer users test (1 par population représentative) :
   ├─ User France/Finance
   ├─ User Commercial
   ├─ User Admin (équipe data)
   └─ User Autre (no access)

   Pour chaque user, pour chaque app :
   ├─ Peut-il voir l'app ? (oui/non/esperé)
   ├─ Données vues = données attendues ? (oui/non)
   ├─ Cross-métier data visible ? (non = bon ✅)
   └─ RLS fonctionne ? (oui/non)

   Exemple test :
   User="France/Finance", App="Finance"
   ├─ Voit l'app ? OUI ✅
   ├─ Voit juste données France+Finance ? OUI ✅
   ├─ Voit données Commercial ? NON ✅
   └─ RESULT : PASS ✅

   User="Commercial", App="Finance"
   ├─ Voit l'app ? NON ✅
   ├─ RESULT : PASS ✅

   User="No Access", App="Finance"
   ├─ Voit l'app ? NON ✅
   ├─ RESULT : PASS ✅
```

**Output** :
```
Testing Results :
├─ 2-3 apps pilot : tested
├─ 48 tests (3 users × 8 apps × 2 scenarios) : XX% PASS
├─ Issues found : [list]
├─ Fixes applied : [list]
└─ Conclusion : "Ready for go-live" ✅
```

---

## TÂCHE 2.3 : Pilot avec Small Audience

**Durée** : 2-3 jours  
**À faire** :

```
✅ PILOT PHASED ROLLOUT :

   Jour 1 : Pilot group 1 (small, tech-savvy)
   ├─ Ajouter 10-20 users à app audience
   ├─ Monitor : feedback ?
   ├─ Issues : quoi ?
   └─ Decision : expand ou fix ?

   Jour 2 : Feedback & fixes
   ├─ Recueillir retours (survey, email)
   ├─ Corriger issues critiques
   ├─ Retest ?

   Jour 3 : Readiness for full go-live
   ├─ "Pilot OK, prêt pour migration complète ?"
   ├─ Checklist : RLS OK ? Apps OK ? Users happy ?
   └─ Decision : GO for Phase 3
```

---

# PHASE 3 : GO-LIVE (Semaine 5)

**Durée** : 7-10 jours  
**Objectif** : Migration complète + support  
**Livrables** : Users on apps, workspaces archived

---

## TÂCHE 3.1 : Formation Users

**Durée** : 1-2 jours (création) + ongoing (sessions)  
**À faire** :

```
✅ FORMATION MATERIEL :

   1. 30-min Video "Apps 101"
      ├─ Où trouver les apps ?
      ├─ Comment chercher mon rapport ?
      ├─ FAQ : bookmarks, favorites, sharing
      └─ Support contact

   2. 1-page Quick Guide
      ├─ Screenshots : avant (workspaces) vs après (apps)
      ├─ Step-by-step : "Trouver mon rapport"
      ├─ Support contact

   3. FAQ Document
      ├─ "Où sont mes workspaces ?"
         └─ Réponse : "Archivés, tout dans apps maintenant"
      ├─ "Vais-je perdre mes données ?"
         └─ Réponse : "Non, RLS conservée, mêmes données"
      ├─ "Pourquoi ce changement ?"
         └─ Réponse : "UX meilleure, navigation plus simple"
      └─ "Qui appeler si problème ?"
         └─ Réponse : "[support email/ticket]"
```

---

## TÂCHE 3.2 : Migration Progressive

**Durée** : 2-3 jours  
**À faire** :

```
✅ PHASED MIGRATION :

   Jour 1 (Jour J) :
   ├─ Apps live (all audiences added)
   ├─ Announcement email sent
   ├─ Support team on standby
   ├─ Monitor adoption

   Jour 2-3 :
   ├─ First 24h issues resolved
   ├─ Users settle in
   ├─ Tech issues triage + fix

   Jour 4-7 (1 semaine post-live) :
   ├─ Weekly check-ins
   ├─ Adoption metrics collected
   ├─ Outstanding issues resolved
   ├─ Training follow-ups as needed
```

---

## TÂCHE 3.3 : Décommissionner Old Workspaces

**Durée** : 1-2 jours  
**À faire** :

```
✅ CLEANUP :

   1. Archive old workspaces (30-day retention)
      └─ Ne pas supprimer immédiatement (rollback possibility)
      └─ Communiquer : "Workspaces archived, apps live"

   2. Documenter mapping (old → new)
      └─ "WS_Finance_1 + WS_Finance_2 = App Finance Reporting"
      └─ Pour référence future + audit trail

   3. Update documentation
      └─ Intranet : "Power BI workspaces fermés, utilisez apps"
      └─ IT handbook : "Comment accéder à BI ?"
```

---

## TÂCHE 3.4 : Post-Live Support (1 semaine)

**Durée** : 3-5 jours  
**À faire** :

```
✅ SUPPORT :

   Week 1 post-live :
   ├─ Hotline actif (email/ticket)
   ├─ Response time : <2h pour critiques
   ├─ Common issues triage :
   │  ├─ "Je vois pas mon app" → check audience
   │  ├─ "Rapport ne charge pas" → RLS issue ?
   │  ├─ "Données différentes" → data validation
   │  └─ "Old workspace ne répond pas" → expected (archived)
   ├─ Escalate to PBI team if needed
   └─ Document solutions (FAQ update)

   After week 1 :
   └─ Support transition to standard IT ticketing
```

---

# RÉSUMÉ EFFORT & TIMELINE

```
Phase 0 : AUDIT          → Semaine 1, 4-6 jours
   ├─ Dérives (2-3j)
   ├─ RLS check (1j)
   ├─ Architecture decision (2-3j)
   └─ Cleanup if needed (1-2j)

Phase 1 : DESIGN         → Semaine 2, 6-11 jours
   ├─ Audiences mapping (3-5j)
   ├─ RLS validation (1-2j)
   ├─ Governance policy (1-2j)
   └─ Communication plan (1j)

Phase 2 : IMPLÉMENTATION → Semaine 3-4, 9-17 jours
   ├─ Create apps (4-6j)
   ├─ Testing (3-5j)
   └─ Pilot (2-3j)

Phase 3 : GO-LIVE        → Semaine 5, 7-10 jours
   ├─ Training (1-2j)
   ├─ Migration (2-3j)
   ├─ Decommission (1-2j)
   └─ Support (3-5j)

════════════════════════════════════════════════════
TOTAL : 26-44 jours = 5-6 semaines = 1.5 mois ⚡
```

---

# RESSOURCES & EFFORT PAR RÔLE

```
Équipe Data (1 person)
├─ Phase 0 : 4-6 jours (audit + decision)
├─ Phase 1 : 4-8 jours (design)
├─ Phase 2 : 6-12 jours (implementation)
├─ Phase 3 : 4-6 jours (go-live + support)
└─ TOTAL : 18-32 jours

Admin/DSI (0.5 person)
├─ Phase 0 : 2 jours (dérives check)
├─ Phase 1 : 1 jour (governance)
├─ Phase 2 : 1-2 jours (testing)
├─ Phase 3 : 2-3 jours (migration support)
└─ TOTAL : 6-8 jours

Métier Validation (0.3 person)
├─ Phase 0 : 1-2 jours (architecture validation)
├─ Phase 1 : 1-2 jours (audiences review)
├─ Phase 2 : 2-3 jours (pilot feedback)
└─ TOTAL : 4-7 jours

════════════════════════════════════════════════════
TOTAL TEAM EFFORT : 28-47 jours
Équipe Data dédication : 50% sur 6 semaines = FAISABLE ✅
```

---

# DÉCISION GATES

```
GATE 1 (Fin Phase 0) : Audit OK ?
├─ ✅ Baseline compris
├─ ✅ Architecture décidée
├─ ✅ Dérives identifiées/nettoyées
└─ Decision : GO for Phase 1 ? → OUI/NON

GATE 2 (Fin Phase 1) : Design OK ?
├─ ✅ Audiences mapping validé
├─ ✅ RLS check passed
├─ ✅ Governance policy signéé
└─ Decision : GO for Phase 2 ? → OUI/NON

GATE 3 (Fin Phase 2) : Implementation OK ?
├─ ✅ Apps created & tested
├─ ✅ Pilot passed
├─ ✅ Users ready
└─ Decision : GO for Phase 3 ? → OUI/NON

GATE 4 (Fin Phase 3) : Migration OK ?
├─ ✅ Users on apps
├─ ✅ Workspaces archived
├─ ✅ Support stable
└─ Decision : Migration DONE ✅
```

---

# CHECKLIST PRÊT À DÉMARRER

```
✅ Phase 0 AUDIT (À faire maintenant)
   □ Accès au fichier Excel workspaces/datasets/populations
   □ Accès à Azure AD (vérifier groupes)
   □ 1 person de l'équipe data dédié (4-6 jours)
   □ 1 admin/DSI pour validation (2 jours)
   □ Réunion architecture (2-3 jours)
   □ Calendrier bloqué Semaine 1

✅ Phase 1-3 (Préparer)
   □ Équipe data disponible 50% sur Semaine 2-5
   □ Support users / champions identifiés
   □ Équipe métier pour validation (light)
   □ PBI tenant access confirmé
   □ Support contacts définis

✅ Communication
   □ Steering committee buy-in
   □ Announcement préparé
   □ FAQ template ready
   □ Training video slot reserved
```

---

# PROCHAINES ÉTAPES

```
✅ TOUT DE SUITE (cette semaine) :
   1. Lire ce document
   2. Confirmer : Migration YES/NO ?
   3. Confirmer : Ressources dispo Phase 0 ?
   4. Bloquer Semaine 1 pour audit

✅ SEMAINE 1 (Phase 0) :
   1. Tâche 0.1 : Audit dérives
   2. Tâche 0.2 : RLS coverage check
   3. Tâche 0.3 : Architecture decision
   4. Tâche 0.4 : Cleanup (si needed)
   5. Gate 1 : GO decision

✅ SEMAINE 2 (Phase 1) :
   1. Tâche 1.1 : Audiences mapping
   2. Tâche 1.2 : RLS validation
   3. Tâche 1.3 : Governance policy
   4. Tâche 1.4 : Communication

✅ SEMAINE 3-4 (Phase 2) :
   1. Tâche 2.1 : Create apps
   2. Tâche 2.2 : Testing
   3. Tâche 2.3 : Pilot

✅ SEMAINE 5 (Phase 3) :
   1. Tâche 3.1 : Formation
   2. Tâche 3.2 : Migration
   3. Tâche 3.3 : Cleanup
   4. Tâche 3.4 : Support
```

---

# QUESTIONS À RÉPONDRE MAINTENANT

```
❓ Architecture
   → "1 app par métier (fusion)" ou "1 app par workspace (1:1)" ?
   → Combien d'apps attendues ?

❓ Ressources
   → Équipe data : 50% disponible pendant 6 semaines ?
   → Admin/DSI : 10-15% disponible ?
   → Métier : 5-10% disponible ?

❓ Dérives actuelles
   → Nb de users individuels en workspace ?
   → Admins non-équipe data ?
   → Populations inactives ?

❓ Priorités post-migration
   → Audit continu requis (hebdo/mensuel) ?
   → Gouvernance stricte ou flexible ?

❓ Timeline
   → Démarrer Phase 0 MAINTENANT (Semaine 1) ?
   → Ou repousser ?
```

---

**Prêt à démarrer Phase 0 AUDIT ?** 🚀
