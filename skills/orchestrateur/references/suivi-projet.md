## Suivi projet partagé (fichiers de collaboration)

Le suivi du projet passe par des fichiers versionnés dans le dépôt. C'est la mémoire commune entre les sessions, les agents et les humains : tout ce qui n'y est pas écrit est perdu à la session suivante.

### Arborescence

```
.projet/
  CONTEXTE.md      principes, stack, conventions, règles qualité (rarement modifié)
  VISION.md        problème, utilisateurs, objectifs, hors-périmètre
  STATUT.yaml      SOURCE DE VÉRITÉ de l'avancement
  JOURNAL.md       une entrée par session (passation)
  QUESTIONS.md     points ouverts et leurs réponses
  REPORTE.md       idées et travaux sortis du périmètre
  decisions/
    README.md      index des décisions
    0001-titre.md  une décision par fichier (ADR)
  features/
    001-nom/
      spec.md      QUOI / POURQUOI + critères d'acceptation
      plan.md      COMMENT (conception, fichiers touchés, risques)
      taches.md    checklist T001, T002…
```

Identifiants : features `001-nom`, tâches `T001` (uniques dans la feature), décisions `0001`, questions `Q001`. Toujours citer par identifiant (`001/T003`, `ADR-0004`, `Q002`).

### Démarrage de session (obligatoire)

Si `.projet/` existe, lire dans cet ordre avant toute action :
1. `CONTEXTE.md`
2. `STATUT.yaml`
3. les 3 dernières entrées de `JOURNAL.md`
4. `QUESTIONS.md` (questions ouvertes seulement)
5. `decisions/README.md`
6. `spec.md`, `plan.md`, `taches.md` de la feature en cours

Puis annoncer : feature en cours, prochaine tâche, questions bloquantes.
Si `.projet/` n'existe pas et que le travail n'est pas trivial : proposer de l'initialiser (ne rien créer sans accord).

### Cycle d'une feature et fichiers écrits

| Étape | Écrit | Condition pour passer à la suite |
|---|---|---|
| Cadrage | `features/NNN/spec.md`, `QUESTIONS.md` | aucune question bloquante ouverte sur la feature |
| Conception | `plan.md`, ADR si décision structurante | plan conforme à `CONTEXTE.md` (sinon ADR qui justifie l'écart) |
| Découpage | `taches.md` | chaque tâche est vérifiable seule |
| Réalisation | cocher `taches.md`, `STATUT.yaml` | tests / build passent pour la tâche |
| Revue | `STATUT.yaml` → `review` puis `done` | critères d'acceptation de `spec.md` vérifiés |
| Clôture de session | `JOURNAL.md` | toujours, même session courte |

Statuts d'une feature : `idee → cadrage → pret → en-cours → review → done` (ou `bloque`, avec la raison).

### Prise de décision

Est une décision à consigner : choix de techno, de stockage, de format d'API ou de données, de structure du code, écart à `CONTEXTE.md`, changement de périmètre, abandon d'une piste.

Procédure :
1. Si plusieurs options sont crédibles, les présenter avec leurs compromis et **attendre l'arbitrage** de l'utilisateur.
2. Créer `decisions/NNNN-titre-court.md` :

   ```markdown
   # NNNN – Titre
   Date : AAAA-MM-JJ
   Statut : Proposée | Acceptée | Remplacée par NNNN
   Décideur : <qui a tranché>
   Concerne : <feature(s) ou « transverse »>

   ## Contexte
   ## Options envisagées
   - A : … (+ / −)
   - B : … (+ / −)
   ## Décision
   ## Conséquences
   ```

3. Ajouter la ligne dans `decisions/README.md` : `| 0004 | Titre | Acceptée | 2026-11-19 | 002-export |`.
4. Citer `ADR-NNNN` dans le `plan.md` concerné.
5. Une décision n'est jamais supprimée ni réécrite : on en crée une nouvelle et on passe l'ancienne en « Remplacée par NNNN ».
6. Une décision « Proposée » ne doit pas être implémentée.

### Formats

`STATUT.yaml` :

```yaml
projet: nom
maj: 2026-11-19
feature_en_cours: 002-export
features:
  001-auth:   { statut: done }
  002-export: { statut: en-cours, taches: "3/7", prochaine: T004 }
  003-notifs: { statut: bloque, raison: "Q004 – choix du fournisseur" }
```

`QUESTIONS.md` :

```markdown
## Q004 – Fournisseur de notifications ? [OUVERTE] (bloque 003)
Posée le 2026-11-18. Contexte : …
→ Réponse (2026-11-19) : … → voir ADR-0005   [RÉSOLUE]
```

`JOURNAL.md` (entrée ajoutée en haut) :

```markdown
## 2026-11-19 – 002-export
Fait : T002, T003
Décisions : ADR-0004
Questions ouvertes : Q004
Prochaine étape : T004 – générer le CSV
Points d'attention : test X instable
```

`taches.md` :

```markdown
- [x] T001 Modèle de données export
- [ ] T004 Générer le CSV — dépend de T003
```

### Règles de collaboration

1. `STATUT.yaml` fait foi. Si `taches.md` ou le code le contredit, signaler l'incohérence au lieu de corriger en silence.
2. Un seul rédacteur par fichier à la fois. Plusieurs agents en parallèle : chacun ne touche que sa feature, et `STATUT.yaml` / `JOURNAL.md` sont mis à jour par l'orchestrateur.
3. Ne jamais cocher une tâche sans preuve (test, build, vérification d'exécution).
4. Ne pas réécrire `spec.md` en cours de réalisation : un changement de périmètre passe par une question ou un ADR, puis une mise à jour explicite de la spec.
5. Tout ce qui sort du périmètre va dans `REPORTE.md`, pas dans le code.
6. Fichiers courts : les détails vont dans la feature, les fichiers racine restent lisibles en 1 minute.

### « Où en est-on ? »

Répondre à partir des fichiers seulement :

```
## Statut – <projet> – <date>
Feature en cours : 002-export (3/7) – prochaine : T004
Bloqué : 003-notifs (Q004)
Dernières décisions : ADR-0004, ADR-0005
Questions ouvertes : Q004
Prochaine étape recommandée : …
```
