# 📊 Évaluation Migration Workspace → Apps Power BI
**Date** : 26/05/2026  
**Contexte** : Orga hiérarchique (France/Finance, Commercial/Équipe, etc.) + RLS par population + ~5 équipe data

---

## 1️⃣ COÛTS & TEMPS ESTIMÉ

### 1.1 Récapitulatif global

| Phase | Durée | Effort (j/h) | Coût (si prestataire) | Notes |
|-------|-------|---|---|---|
| **Phase 1 : Assessment** | 2-3 sem | 15-20 jh | €3-5K | Audit état actuel |
| **Phase 2 : Design** | 3-4 sem | 25-30 jh | €5-8K | Gouvernance + taxonomie apps |
| **Phase 3 : Pilot (2-3 apps)** | 6-8 sem | 40-50 jh | €8-12K | Création + test + formation |
| **Phase 4 : Rollout progressif** | 12-16 sem | 60-80 jh | €12-20K | Migration 8+ workspaces |
| **Phase 5 : Stabilisation** | 4-6 sem | 20-30 jh | €4-8K | Support, corrections, optimisations |
| **TOTAL** | **6-9 mois** | **160-210 jh** | **€32-53K** | Si externe |

---

## 2️⃣ DIFFICULTÉS MAJEURS À ANTICIPER

### 2.1 Risques organisationnels

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|-----------|
| **Résistance users (perte de flexibilité)** | 🔴 Haute | 🔴 Haut | Communication précoce + workshops |
| **Goulot App Owner (publier les apps)** | 🟠 Moyen | 🟠 Moyen | Multi-ownership ou délégation |
| **Shadow IT persistant** | 🟠 Moyen | 🔴 Haut | Bloquer workspaces, audit strict |
| **RLS cassée après migration** | 🔴 Haute | 🔴 Haut | Test exhaustif avant production |

---

## 3️⃣ ÉTAPES CLÉS (Résumé)

```
PHASE 1 : Assessment         → 2-3 sem (audit baseline + dérives)
PHASE 2 : Design             → 3-4 sem (gouvernance + taxonomie)
PHASE 3 : Pilot (2-3 apps)   → 6-8 sem (test + formation owners)
PHASE 4 : Rollout (5+ apps)  → 12-16 sem (vagues progressives)
PHASE 5 : Stabilisation      → 4-6 sem (ops handover)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL : 6-9 mois
```

---

## 4️⃣ IMPACT SUR L'AUDIT (PRIO 1)

### Avant migration (Workspace Direct)
- **Audit complexity** : 🔴 HAUTE
- **Run time** : 2h 45min
- **Manual work** : 30%
- **Findings** : 20-30 issues

### Après migration (Apps)
- **Audit complexity** : 🟢 BASSE
- **Run time** : 15 min ⚡
- **Manual work** : 5%
- **Findings** : 0-2 issues
- **Automation** : 95%

**Gain audit** : -95% temps, +4x fréquence, 100% automation

---

## 5️⃣ ESTIMATION BUDGET

| Composante | Coût |
|---|---|
| Interne (6-8.5 ETP × €1000/j) | €120-170K |
| Coaching externe (40j) | €8-12K |
| Outils/licenses | €2-5K |
| Training/comms | €1-2K |
| **TOTAL** | **€131-189K** |

**ROI** : 4-5 ans (compliance/risk reduction = immédiat ✅)

---

## 6️⃣ FACTEURS CRITIQUES DE SUCCÈS

```
✅ Executive sponsorship (budget + visible support)
✅ Clear governance policy (avant pilot)
✅ Strong app owners (trained + motivated)
✅ RLS fully validated (test matrix signed)
✅ User adoption strategy (early involvement + training)
✅ Audit backbone (PRIO 1 baseline first)
✅ Communication cadence (weekly updates)
✅ Risk management (active mitigations)
```

---

## 7️⃣ DÉCISION GATES

**Gate 1 (Phase 1→2)** : Assessment complet ? ✅ Dérives identifiées ? Baseline clean ?

**Gate 2 (Phase 2→3)** : Design approuvé ? ✅ Gouvernance policy ? RLS test matrix ?

**Gate 3 (Phase 3→4)** : Pilot OK ? ✅ 3 apps en prod ? Adoption ≥60% ?

**Gate 4 (Phase 4→5)** : Full rollout done ? ✅ Adoption ≥80% ? Ops ready ?

---

## 8️⃣ RECOMMANDATION FINALE

```
✅ START PRIO 1 (Audit) NOW
   └─ 4-6 weeks (Phase 1 Assessment)
   └─ Outputs : Dashboard + findings

⏳ DECIDE PRIO 2 après audit complet
   ├─ IF dérives min + governance OK → GO
   └─ IF trop de dérives → Cleanup d'abord

✅ EXECUTE si GO decision
   └─ 6-9 mois (phases pilot + waves + stabilisation)
```

**Migration est FEASIBLE & RECOMMANDÉE SI :**
- ✅ Strong app owners + governance
- ✅ RLS fully tested before pilot
- ✅ Executive commitment (6-9 mois)
- ✅ Audit foundation (PRIO 1) in place

---

## 📌 DOCUMENT COMPLET

Ce fichier est un **résumé exécutif**. Pour la version détaillée avec :
- Étapes exhaustives (28 sous-étapes)
- Queries SQL pour audit
- Tableaux comparatifs avant/après
- Risk register détaillé
- Templates gouvernance

→ Demande la **version complète** (40+ pages)

