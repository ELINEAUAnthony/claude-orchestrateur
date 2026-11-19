# Fin : archivage des sessions

Les fichiers `.projet/` ne s'archivent pas : ils sont l'historique versionné du projet (feature `done` dans `STATUT.yaml`).

Un seul AskUserQuestion, `multiSelect: true` :

- **Les sous-sessions des tâches (Recommandé)** — seulement si `execution: sous-sessions` : les sessions `local_…` notées dans `taches.md`.
- **Les anciennes sessions principales** — seulement s'il y a eu des passations (`HANDOFF-<n>.md`) : les sessions `orchestrateur-<n>` sauf la session courante.
- **Réactiver Vercel** — seulement si `vercel: desactive` dans `STATUT.yaml` (la coupure est aussi le réglage par défaut des nouvelles sessions).

Rien de coché → rien à faire.

- Sessions : `mcp__ccd_session_mgmt__list_sessions` puis `mcp__ccd_session_mgmt__archive_session` pour chacune ; jamais une session encore active. Une session archivée reste récupérable.
- Vercel : `mcp__ccd_connectors__set_session_connector_enabled` (`connector: "Vercel"`, `enabled: true`).

Termine par une ligne récapitulant ce qui a été archivé.
