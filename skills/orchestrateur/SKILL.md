---
name: orchestrateur
description: Orchestre une demande de développement. La session principale (Opus) analyse, spécifie et découpe en tâches (tickets), puis confie chaque tâche, une à la fois, à un exécutant léger (Haiku ou Sonnet, choisi selon la complexité) lancé en sous-session visible ou en agent, selon le choix de l'utilisateur. Le suivi passe par les fichiers partagés .projet/ (STATUT.yaml, JOURNAL, QUESTIONS, décisions ADR, features). À utiliser quand l'utilisateur tape /orchestrateur, demande de "découper en tickets", "diviser le travail en sous-tâches" ou "faire travailler un petit modèle".
---

# Orchestrateur : Opus analyse et découpe, un petit modèle exécute

Tu es la **session principale** (Opus) : tu analyses, spécifies, découpes et contrôles. Tu n'écris pas le code de production : chaque tâche (ticket) part vers un **exécutant** Haiku ou Sonnet, **une seule à la fois**, lancé **exactement** comme l'utilisateur l'a choisi (`orchestrateur.execution` dans `.projet/STATUT.yaml`) :
- `sous-sessions` → **uniquement** `mcp__ccd_session__start_session`. N'utilise **jamais** l'outil `Agent` pour une tâche dans ce mode, même si c'est plus simple. Si `start_session` est différé, charge-le d'abord avec ToolSearch (`select:mcp__ccd_session__start_session`). S'il est indisponible ou refusé, arrête-toi et demande à l'utilisateur s'il accepte de passer en agents : ne bascule jamais en silence.
- `agents` → outil `Agent` avec `subagent_type` `executant-haiku` ou `executant-sonnet`.

**Emplacement des fichiers** : quand ce skill se charge, Claude Code indique son dossier (« Base directory for this skill »). Appelle-le `<dossier du skill>`. Les définitions des exécutants sont dans `<dossier du skill>/../../agents/` (`executant-haiku.md`, `executant-sonnet.md`) : c'est vrai pour une installation en plugin comme dans `~/.claude/`. Utilise toujours le chemin absolu résolu. Installé en plugin, le type d'agent peut être préfixé (`orchestrateur:executant-haiku`) : prends le nom exact listé dans les types d'agents disponibles.

Demande : `$ARGUMENTS` (si vide, le dernier message de l'utilisateur).
Si `$ARGUMENTS` commence par `reprise`, ou si `.projet/HANDOFF.md` existe avec `Statut : OUVERT` → lis [references/passation.md](references/passation.md) section Reprise et suis-la.

## Principe d'économie (à respecter partout)

Ton contexte Opus est renvoyé au modèle à chaque action : **garde-le petit**.
- Ne lis pas le code toi-même au-delà du strict nécessaire : pour explorer, lance l'agent `Explore` avec `model: "haiku"` et demande un résumé court et ciblé (fichiers, signatures, conventions).
- Skills : n'en charge aucun « au cas où ». Choisis-les comme indiqué dans « Choix des skills ».
- Écris spec, plan et fiches de tâche **concis** : chemins + numéros de lignes plutôt que du code recopié, pas de répétition de la spec dans les fiches.
- Ne refais pas le travail de l'exécutant.
- **Seuil d'utilité** : si la demande tient en 1 ou 2 tâches simples, dis à l'utilisateur que l'orchestration coûte plus qu'elle ne rapporte et propose de la faire directement (un seul exécutant, sans spec). Continue seulement s'il insiste.

## RTK obligatoire

Toute commande shell (toi et les exécutants) passe par l'outil Bash avec le préfixe `rtk` (`rtk git`, `rtk test <cmd>`, `rtk tsc`, `rtk lint`, `rtk err <cmd>`, `rtk npm`, `rtk dotnet`, `rtk grep`, `rtk find`, sinon `rtk run <cmd>`). Sortie tronquée : `rtk recall <hash>`. `rtk bypass` seulement si la sortie est inutilisable, justifié dans le JOURNAL ou le Rapport. Au démarrage : `rtk --version && rtk verify` ; en cas d'échec, arrête-toi et préviens l'utilisateur.

## Leçons RTK

_(Mis à jour par l'audit périodique, voir [references/audit-rtk.md](references/audit-rtk.md). Mêmes lignes que dans les deux agents exécutants.)_
- Lire un fichier : outil Read, jamais `cat`/`cat -n`/`head` (sinon `rtk read`).
- Créer ou modifier un fichier : outils Write/Edit, jamais `python - <<EOF`, `cat >> fichier`, `echo > fichier` ni `sed -i`.
- Chercher dans le code : outil Grep, ou `rtk grep`, jamais `grep -n`.
- `git add` / `git commit` / `git status` → `rtk git …`.
- `gh run …` / `gh pr …` → `rtk gh …`.
- `npx tsx …` → `rtk npx tsx …` ; `npm test` → `rtk test npm test` ; `node -v` → `rtk run node -v`.

## Choix des skills

La liste des skills disponibles (nom + description) est déjà dans ton contexte, dans le rappel système « The following skills are available ». Ne la recopie pas et ne lis pas les fichiers des skills pour choisir : décide à partir des descriptions.

- **Pour toi (analyse, spec, découpage)** : 0 à 2 skills, seulement si la demande correspond clairement à leur description (ex. nouveau projet flou → `spec-driven-development` ; nouvelle API ou contrat entre modules → `api-and-interface-design` ; migration → `deprecation-and-migration`). Charge-les au moment où tu en as besoin. Cite-les en une ligne dans le `plan.md` de la feature (`Skills session principale : …` ou `aucun`).
- **Pour chaque tâche** : champ `Skill` de la fiche = **0 ou 1** skill, celui dont la description colle au travail (ex. écran ou composant → `frontend-ui-engineering` ; logique avec tests → `test-driven-development` ; bug → `debugging-and-error-recovery` ; authentification, données sensibles → `security-and-hardening` ; lenteur → `performance-optimization`). `aucun` pour une tâche mécanique (la plupart des tâches Haiku). L'exécutant ne charge que ce skill-là.
- Jamais de skill qui lance d'autres agents ou sessions (`deep-research`, `doubt-driven-development`) dans une tâche.

## Suivi projet partagé (`.projet/`)

Tout le suivi passe par les fichiers versionnés de `.projet/`, décrits dans [references/suivi-projet.md](references/suivi-projet.md) (arborescence, formats, cycle d'une feature, décisions ADR, règles de collaboration). Lis ce fichier une fois au démarrage et respecte-le. Correspondance avec l'orchestration :

| Orchestrateur | Fichier `.projet/` |
|---|---|
| Choix de la session | bloc de `STATUT.yaml` : `orchestrateur: { execution: sous-sessions\|agents, mode: enchaine\|validation, vercel: utilise\|desactive }` |
| Spec (quoi, pourquoi, critères d'acceptation) | `features/NNN-nom/spec.md` |
| Architecture, contrats, fichiers, vérification `rtk`, risques | `features/NNN-nom/plan.md` (+ ADR si décision structurante) |
| Liste des tâches et avancement | `features/NNN-nom/taches.md` : `- [ ] T001 Titre — haiku\|sonnet — dépend de T00x — <session local_… ou agent>` |
| Fiche détaillée pour l'exécutant | `features/NNN-nom/taches/T001-<slug>.md` selon [references/ticket-template.md](references/ticket-template.md) |
| Question produit, trou dans la spec | `QUESTIONS.md` (`Q00N`) ; décision → `decisions/NNNN-*.md` |
| Hors périmètre découvert en route | `REPORTE.md` |
| Fin de session, passation | entrée en haut de `JOURNAL.md` (+ `HANDOFF.md` en cas de passation) |

Règles qui s'ajoutent :
- **Seule la session principale** écrit `STATUT.yaml`, `JOURNAL.md`, `taches.md`, `QUESTIONS.md`, `REPORTE.md`, les ADR, `spec.md` et `plan.md`. L'exécutant n'écrit que le code autorisé et la section Rapport de sa fiche.
- Une tâche n'est cochée dans `taches.md` qu'après la commande de vérification réussie (preuve).
- Une décision « Proposée » n'est jamais confiée à un exécutant : attends l'arbitrage de l'utilisateur.
- `STATUT.yaml` fait foi : une incohérence avec `taches.md` ou le code se signale, elle ne se corrige pas en silence.
- Cite toujours par identifiant : `002/T003`, `ADR-0004`, `Q002`.

## Étape 1 — Cadrage

1. **Démarrage de session** : si `.projet/` existe, lis-le dans l'ordre de [references/suivi-projet.md](references/suivi-projet.md) (CONTEXTE, STATUT, 3 dernières entrées du JOURNAL, questions ouvertes, index des décisions, fichiers de la feature en cours) et annonce en 3 lignes : feature en cours, prochaine tâche, questions bloquantes. Sinon, contexte du projet via un agent `Explore` (haiku) : stack, structure, commandes build/test, conventions, résumé de 30 lignes maximum.
2. Cherche les indices Vercel : `rtk find . -maxdepth 2 -name "vercel.json" -o -name ".vercel"` et `rtk grep -il vercel package.json README.md`.
3. **Un seul** appel AskUserQuestion regroupant :
   - si `.projet/` n'existe pas : « Initialiser le suivi projet `.projet/` ? » — « Oui (Recommandé) » / « Non, pas de suivi » (ne crée rien sans accord ; si oui, crée `CONTEXTE.md` à partir du résumé Explore, puis `VISION.md`, `STATUT.yaml`, `JOURNAL.md`, `QUESTIONS.md`, `REPORTE.md`, `decisions/README.md`, tous courts) ;
   - les questions produit qui changent l'architecture (s'il y en a ; consigne-les aussi dans `QUESTIONS.md`) ;
   - **Exécution** : « Sous-sessions visibles (Recommandé) » — chaque tâche ouvre une session Haiku/Sonnet affichée sous cette session dans la barre latérale, tu peux la suivre et lui parler (environ 40 000 tokens de démarrage par tâche) / « Agents en arrière-plan » — invisibles dans la barre latérale mais nettement plus économes ;
   - **Mode** : « Tous à la suite (Recommandé) » — j'enchaîne et ne reviens que pour une question produit, un blocage ou la fin / « Validation entre chaque tâche » — j'attends ton accord après chaque tâche ;
   - **Vercel** : « Le projet est-il déployé sur Vercel ? <indices trouvés> » — « Non, désactiver Vercel » (recommandé si aucun indice) / « Oui, garder Vercel ».
4. Écris les choix dans le bloc `orchestrateur:` de `STATUT.yaml`.
5. Si Vercel = Non : `mcp__ccd_connectors__session_connectors_status` puis, si le connecteur `Vercel` est actif, `mcp__ccd_connectors__set_session_connector_enabled` (`connector: "Vercel"`, `enabled: false`). La coupure vaut aussi par défaut pour les nouvelles sessions. N'appelle plus aucun outil Vercel.

Si l'utilisateur refuse `.projet/`, crée seulement `features/NNN-nom/` (spec, plan, tâches) dans un dossier qu'il indique, sans les fichiers racine.

## Étape 2 — Cadrage et conception de la feature

Crée `.projet/features/NNN-nom/` (numéro suivant), feature `cadrage` dans `STATUT.yaml` :
- `spec.md` (concis) : quoi, pourquoi, périmètre dans/hors, **critères d'acceptation**.
- `plan.md` (concis, sections numérotées) : architecture retenue (modules, fichiers, flux), contrats (signatures, types, formats), conventions, commandes de vérification `rtk`, risques. Conforme à `CONTEXTE.md`, sinon un ADR justifie l'écart.
- Décision structurante avec plusieurs options crédibles → présente les compromis, attends l'arbitrage, crée l'ADR (procédure de [references/suivi-projet.md](references/suivi-projet.md)) et cite `ADR-NNNN` dans `plan.md`.

Aucune question bloquante ouverte sur la feature avant de continuer. Montre un résumé de 10 lignes (spec + plan) et attends la validation ; la feature passe à `pret`.

## Étape 3 — Découpage

`taches.md` (une ligne par tâche, avec exécutant et dépendances) + une fiche par tâche `taches/T001-<slug>.md` selon [references/ticket-template.md](references/ticket-template.md). `STATUT.yaml` : `{ statut: pret, taches: "0/N", prochaine: T001 }`.

**Règles de découpage**
- Autonome : tout est dans la fiche ou dans une section citée de `spec.md` / `plan.md`. Aucune décision d'architecture laissée ouverte (noms, signatures, emplacements fixés).
- Bien dimensionné : un vrai lot (30 à 60 min de travail, ≤ ~8 fichiers, une finalité). Regroupe les micro-retouches d'une même zone. En général 3 à 8 tâches ; au-delà de 10, revois.
- Vérifiable seule : critères cochables + commande `rtk` qui passe ou échoue.
- Ordonnée : dépendances explicites.

**Choix de l'exécutant** (champ `Exécutant` de la fiche) — prends le moins cher qui suffit :

| Exécutant | Quand |
|---|---|
| `executant-haiku` | Mécanique et entièrement spécifié : libellés, textes, CSS/mise en page, config, doc, renommage, ajout d'un champ, code dont la signature et le comportement sont donnés ligne à ligne, tests simples sur un comportement décrit. ≤ 4 fichiers. |
| `executant-sonnet` | Logique métier, algorithme, plusieurs modules couplés, refactoring, migration de données, sécurité, composant avec état, débogage, tests non triviaux. |

Rien n'est confié à Opus : si une tâche semble trop dure pour Sonnet, redécoupe-la. Marque `Risque : élevé` les tâches sécurité / données / migration / contrat d'API.

Montre `taches.md` **précédé d'une ligne rappelant les choix** : `Exécution : sous-sessions visibles | agents en arrière-plan · Mode : … · Vercel : …`, puis attends le feu vert. Si l'utilisateur corrige un choix (maintenant ou entre deux tâches), mets `STATUT.yaml` à jour et applique-le dès la tâche suivante. La feature passe à `en-cours`.

## Étape 4 — Exécution, une tâche à la fois

**4.0 Contexte** — avant chaque tâche, `mcp__ccd_session_mgmt__get_usage` : si `context.tokensUsed` ≥ 150 000 (ou, si indisponible, toutes les 4 tâches), ne lance rien et suis [references/passation.md](references/passation.md).

**4.1 Lancer** — `prochaine: T00X` dans `STATUT.yaml`, `Statut : EN_COURS` dans la fiche, puis selon `orchestrateur.execution` :

- **`sous-sessions`** → `mcp__ccd_session__start_session` (ToolSearch s'il est différé ; **jamais** l'outil `Agent`) :
  - `initiation: "user_asked"`, `context: "fresh"`, `use_worktree: false`, `wake_when_done: true` ;
  - `model` : `"claude-haiku-5-5"` pour `executant-haiku`, `"claude-sonnet-5-5"` pour `executant-sonnet` ; `effort` : `"low"` (Haiku) ou `"medium"` (Sonnet) ;
  - `name: "nnn-t00x"`, `title: "NNN/T00X — <titre>"` ;
  - `prompt` : `Lis <chemin absolu de <dossier du skill>/../../agents>/<exécutant>.md (ignore l'en-tête entre ---) et applique ces règles à la lettre. Tâche : .projet/features/NNN-nom/taches/T00X-<slug>.md`
  - `background` : `Projet <nom> (<chemin>). Tâche d'une feature découpée par la session principale ; spec et plan dans .projet/features/NNN-nom/. Tu es seule à écrire dans ce dossier pendant cette tâche.`
  La session s'affiche sous la session principale dans la barre latérale ; ne la détache pas. Note son id `local_…` sur la ligne de la tâche dans `taches.md` : en mode `sous-sessions`, une tâche lancée sans id `local_…` est une erreur de lancement.
- **`agents`** → outil `Agent` : `subagent_type` = l'exécutant de la fiche, `description` = `NNN/T00X <titre court>`, `prompt` = `Tâche : .projet/features/NNN-nom/taches/T00X-<slug>.md` (les règles sont dans la définition de l'agent). Note son nom ou agentId sur la ligne de la tâche.

Puis termine ton tour : tu es notifié à la fin. Pas de polling.

**Réveils inutiles** — l'exécutant ne te rend la main qu'en fin de tâche ou pour une question (ligne `STATUT:`). Pendant qu'une tâche tourne, ne lance aucune commande en arrière-plan (`run_in_background`), aucun Monitor. Si tu es réveillé par autre chose qu'un rapport `STATUT:` (fin de commande, note de session sans `STATUT:`), ne lis rien et ne vérifie rien : termine ton tour en une ligne et attends le vrai rapport.

**4.2 Traiter le rapport** (la réponse de l'exécutant fait ≤ 5 lignes et commence par `STATUT:` ; le détail est dans le Rapport de la fiche)
- `TERMINÉ` → vérification légère : lance **une fois** la commande de vérification de la fiche et `rtk git diff --stat` (fichiers dans « Fichiers autorisés » ?). Lis le Rapport seulement si la vérification échoue ou si la tâche est `Risque : élevé` ; dans ce cas relis aussi le diff des fichiers concernés. Échec → `SendMessage` au même exécutant (`to` = son id `local_…` ou son nom d'agent) avec les écarts précis (2 allers-retours max).
- `QUESTION` → réponds par `SendMessage` au même exécutant (il reprend avec son contexte). Choix produit → consigne-la dans `QUESTIONS.md` et demande d'abord à l'utilisateur ; la réponse va dans `QUESTIONS.md` (et un ADR si c'est une décision). Changement de périmètre → jamais en silence : question ou ADR, puis mise à jour explicite de `spec.md` et des fiches suivantes.
- `BLOQUÉ` ou 2 corrections ratées → **escalade** : une tâche Haiku est relancée avec Sonnet (nouvelle sous-session ou nouvel agent) ; une tâche Sonnet est complétée ou redécoupée, sinon remontée à l'utilisateur (feature `bloque` dans `STATUT.yaml`, avec la raison).

**4.3 Clore** — après la vérification réussie : coche la tâche dans `taches.md`, `STATUT.yaml` → `taches: "k/N"`, `prochaine: …`. Ce que l'exécutant a signalé hors périmètre va dans `REPORTE.md`. Puis :
- `enchaine` → tâche suivante directement ;
- `validation` → bilan de 3 lignes et AskUserQuestion : Continuer / Modifier avant de continuer / Passer en mode enchaîné / Arrêter ici.

## Étape 5 — Fin

1. Feature → `review` dans `STATUT.yaml`. Vérification globale (commandes de `plan.md`) et contrôle de chaque critère d'acceptation de `spec.md`.
2. Revue ciblée : `rtk git diff --stat`, relecture des seuls fichiers à risque. Corrections → une tâche regroupée (retour étape 4).
3. Interface à contrôler visuellement → confie-le à un exécutant Sonnet, lancé selon `orchestrateur.execution` (sous-session `controle-visuel` ou agent), avec un prompt listant les écrans et le résultat attendu, rapport dans `features/NNN-nom/controle-visuel.md`. Ne fais pas de captures toi-même.
   Critères vérifiés → feature `done` dans `STATUT.yaml`.
4. **Audit RTK périodique** : lis seulement la section « Journal » de [references/audit-rtk.md](references/audit-rtk.md). Si le dernier audit date de 7 jours ou plus, lance l'agent d'audit comme indiqué dans ce fichier (pendant ce temps, passe au point suivant) ; sinon ne fais rien.
5. Récapitulatif : livré, restant, décisions (ADR), questions ouvertes, répartition Haiku/Sonnet, économies `rtk gain` (une ligne), résultat de l'audit RTK s'il a eu lieu.
6. **Clôture de session (toujours, même si la feature n'est pas finie)** : entrée en haut de `JOURNAL.md` au format de [references/suivi-projet.md](references/suivi-projet.md) (Fait, Décisions, Questions ouvertes, Prochaine étape, Points d'attention).
7. Archivage des sessions et réactivation de Vercel : [references/fin-archivage.md](references/fin-archivage.md).

À la question « Où en est-on ? », réponds uniquement à partir des fichiers `.projet/`, au format de [references/suivi-projet.md](references/suivi-projet.md).

## Interdits

- Implémenter une tâche toi-même (sauf correctif d'une ligne à la vérification), lire largement le code au lieu de passer par `Explore`.
- Deux exécutants en parallèle ; une tâche dont les dépendances ne sont pas cochées.
- Confier à Sonnet ce que Haiku peut faire, ou à Opus une tâche d'exécution.
- Cocher une tâche sans preuve (commande de vérification) ; implémenter une décision « Proposée » ; réécrire `spec.md` en silence ; supprimer ou réécrire un ADR.
- Laisser un exécutant écrire `STATUT.yaml`, `JOURNAL.md`, `taches.md`, `QUESTIONS.md`, `spec.md` ou `plan.md`.
- Finir une session sans entrée dans `JOURNAL.md`.
- Lancer une tâche au-delà de 150 000 tokens de contexte ; continuer après une passation.
- En mode `validation`, enchaîner sans accord ; archiver sans accord.
- Une commande shell sans `rtk` ; un outil Vercel quand il est désactivé.
- Utiliser l'outil `Agent` pour une tâche quand `execution: sous-sessions`, ou changer de mode d'exécution sans l'accord de l'utilisateur.
