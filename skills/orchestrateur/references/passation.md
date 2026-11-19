# Passation de la session principale

À lire seulement quand le contexte atteint 150 000 tokens, ou en mode reprise.

## Faire la passation

Jamais pendant qu'un exécutant travaille : attends sa notification, traite son rapport (y compris une QUESTION), puis passe la main. Un exécutant ne rend compte qu'à la session qui l'a lancé.

1. Mets `.projet/` à jour : `STATUT.yaml`, `taches.md`, `QUESTIONS.md`, ADR éventuels. C'est la mémoire principale de la reprise.
2. Ajoute l'entrée de session en haut de `JOURNAL.md` (format de [suivi-projet.md](suivi-projet.md)).
3. Écris `.projet/HANDOFF.md` selon [handoff-template.md](handoff-template.md) : seulement ce que `.projet/` ne contient pas déjà.
4. `mcp__ccd_session__start_session` :
   - `initiation: "user_asked"`, `context: "fresh"`, `use_worktree: false`, sans `model` (hérite d'Opus) ;
   - `name: "orchestrateur-<n>"`, `title: "Orchestrateur (suite) — <projet>"` ;
   - `prompt` : `/orchestrateur reprise` + « Lis `.projet/HANDOFF.md` puis `.projet/` dans l'ordre du démarrage de session, et reprends à NNN/T00X. » ;
   - `background` : objectif de la feature en 2-3 phrases, chemin du projet, « le suivi est dans `.projet/`, `STATUT.yaml` fait foi ».
5. **Rends-la indépendante tout de suite** : `mcp__ccd_session_mgmt__detach_session` avec son `session_id`. La nouvelle session principale doit être une session de premier niveau dans la barre latérale, pas une sous-session de l'ancienne : sinon ses propres sous-sessions de tâches se retrouvent imbriquées deux niveaux plus bas et l'utilisateur ne les voit plus. Si le détachement est refusé, dis-le à l'utilisateur (il peut faire « Detach to top level » depuis la barre latérale).
6. Préviens l'utilisateur en une ligne avec le lien de la nouvelle session, puis arrête-toi.

## Reprise (nouvelle session principale)

1. `rtk --version && rtk verify`. Puis `mcp__ccd_session_mgmt__get_session` (`"self"`) : si elle a encore un `parentSessionId` sans `detached: true`, appelle `mcp__ccd_session_mgmt__detach_session` avec `"self"`.
2. Lis `HANDOFF.md`, puis `.projet/` dans l'ordre du démarrage de session ([suivi-projet.md](suivi-projet.md)). Reprends le bloc `orchestrateur:` de `STATUT.yaml` sans redemander ; si `vercel: desactive` et que le connecteur est actif, coupe-le.
3. Une tâche `EN_COURS` sans Rapport rempli repasse à `A_FAIRE` (l'exécutant de l'ancienne session ne te répondra pas).
4. `HANDOFF.md` → `Statut : CONSOMMÉ`, renomme-le `HANDOFF-<n>.md`.
5. Reprends l'étape 4 du skill. Nouvelles tâches : numérotation à la suite dans `taches.md`.
