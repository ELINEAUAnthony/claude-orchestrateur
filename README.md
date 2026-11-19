# Orchestrateur (plugin Claude Code)

Opus analyse, spécifie et découpe une demande de développement en tâches. Chaque tâche est exécutée une à une par Haiku ou Sonnet (choisi selon la complexité), en sous-session visible dans la barre latérale ou en agent en arrière-plan. Le suivi passe par des fichiers partagés dans `.projet/` (statut, journal, questions, décisions ADR, features).

## Contenu

- `skills/orchestrateur/` : le skill `/orchestrateur` et ses références (modèle de tâche, passation, archivage, audit RTK, suivi projet).
- `agents/executant-haiku.md`, `agents/executant-sonnet.md` : les deux exécutants.

## Prérequis

- **Claude Code**, de préférence dans l'application de bureau : le mode « sous-sessions visibles » et la passation automatique utilisent les outils de sessions de l'application. En ligne de commande seule, choisir le mode « agents ».
- **RTK** installé et branché (`rtk --version`, `rtk verify`) : toutes les commandes shell passent par RTK. Sans RTK, le skill s'arrête au démarrage.
- Accès aux modèles Opus (session principale), Sonnet et Haiku.

## Installation

Depuis GitHub :

```
/plugin marketplace add ELINEAUAnthony/claude-orchestrateur
/plugin install orchestrateur@orchestrateur
```

Depuis un dossier local ou partagé :

```
/plugin marketplace add C:\chemin\vers\claude-orchestrateur
/plugin install orchestrateur@orchestrateur
```

Redémarrer Claude Code ensuite. Si une copie personnelle existe déjà dans `~/.claude/skills/orchestrateur` et `~/.claude/agents/executant-*.md`, la supprimer pour éviter les doublons.

## Utilisation

```
/orchestrateur <ta demande>
```

Au démarrage, une seule question regroupe : initialisation de `.projet/`, questions produit, mode d'exécution (sous-sessions ou agents), enchaînement (tout à la suite ou validation entre chaque tâche), usage de Vercel. Quand le contexte de la session principale dépasse 150 000 tokens, elle passe la main à une nouvelle session principale (`/orchestrateur reprise`).

## Mise à jour

Augmenter `version` dans `.claude-plugin/plugin.json`, pousser, puis chez les utilisateurs :

```
/plugin marketplace update orchestrateur
```

Les leçons RTK apprises localement par l'audit périodique sont écrasées à chaque mise à jour.
