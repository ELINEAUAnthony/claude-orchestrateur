# Modèle de passation (`.projet/HANDOFF.md`)

Complète `.projet/` sans le répéter : l'avancement est dans `STATUT.yaml` et `taches.md`, les décisions dans les ADR, les questions dans `QUESTIONS.md`, le résumé de session dans `JOURNAL.md`. Ici, seulement ce qui ne figure nulle part ailleurs. Moins de 30 lignes.

```markdown
# Passation n°<n> — <projet>

**Statut** : OUVERT
**Date** : <AAAA-MM-JJ HH:MM>
**Session sortante** : <titre> (`local_…`), contexte à <xxx> 000 tokens
**Reprendre à** : NNN/T00X

## Tâche en cours
NNN/T00Y — exécutant, ce qui reste (repasse à A_FAIRE si son Rapport est vide), ou « aucune ».

## Réponses déjà données aux exécutants
- NNN/T00X : « … » → « … »

## Consignes de l'utilisateur non écrites ailleurs
- …

## Première action attendue
<ex : relancer 002/T004 avec Sonnet, puis lancer 002/T005>
```
