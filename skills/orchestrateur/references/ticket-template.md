# Modèle de ticket

Une fiche par tâche : `.projet/features/NNN-nom/taches/T001-<slug>.md`. **Concise** : des chemins et numéros de lignes plutôt que du code recopié, pas de répétition de `spec.md` / `plan.md` (cite la section). Visé : 1 500 à 4 000 caractères. Une fiche Haiku est plus explicite (étapes ligne à ligne), une fiche Sonnet peut décrire le résultat attendu.

```markdown
# T-XXX — <Titre court à l'impératif>

**Statut** : A_FAIRE | EN_COURS | QUESTION | BLOQUÉ | TERMINÉ
**Exécutant** : executant-haiku | executant-sonnet
**Skill** : <nom exact d'un skill disponible> | aucun
**Dépend de** : T-00Y (ou Aucun)
**Risque** : normal | élevé
**Référence** : `spec.md` § <n>, `plan.md` § <n>, ADR-NNNN éventuel

## Objectif
Une phrase.

## Contexte
- Code existant : `chemin/fichier.ext:L10-L40` (ce qu'il fait en une ligne)
- Déjà livré par les tickets précédents : …

## Travail à faire
1. Créer `src/x/y.ts` : `nomFonction(arg: Type): Retour` qui …
2. Modifier `src/z.ts:L52` : appeler `nomFonction` après …
3. Tests `tests/x/y.test.ts` : cas nominal, vide, erreur.

## Contrats
Signatures, types, formats, noms exacts (ou « voir SPEC § n »).

## Fichiers autorisés
- `src/x/y.ts` (création), `src/z.ts` (modification), `tests/x/y.test.ts` (création)

## Hors périmètre
- …

## Critères d'acceptation
- [ ] …
- [ ] La vérification passe

## Vérification
`rtk <commande exacte>` (ex : `rtk vitest run tests/x/y.test.ts && rtk lint`)

## Rapport
_(Rempli par l'exécutant.)_
- STATUT :
- Fichiers modifiés :
- Vérification :
- RTK : bypass utilisés (commande + raison) ou « aucun »
- Question / blocage / remarques :
```

## Check-list avant lancement

- [ ] Faisable sans aucune question d'architecture ; tous les noms sont fixés.
- [ ] Lot de 30 à 60 min, ≤ ~8 fichiers, une finalité ; micro-retouches regroupées.
- [ ] Exécutant le moins cher qui suffit (Haiku si mécanique et ≤ 4 fichiers).
- [ ] Skill : 0 ou 1, seulement s'il colle vraiment au travail du ticket.
- [ ] Vérification = commande `rtk` exécutable.
- [ ] Dépendances `TERMINÉ`.
