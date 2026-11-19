# Audit RTK périodique

But : améliorer en continu les règles RTK du skill et des exécutants à partir de l'historique réel, sans y passer des tokens à chaque fois.

## Quand

Seulement à l'étape 5 (fin d'orchestration), et seulement si le dernier audit du journal ci-dessous date de **7 jours ou plus**. Sinon, ne fais rien et ne lis pas ce fichier plus loin que le journal.

La session principale ne fait pas l'audit elle-même : elle lance l'outil `Agent` avec `subagent_type` = l'exécutant Sonnet (`executant-sonnet` ou `orchestrateur:executant-sonnet`), `description: "Audit RTK"`, `prompt: "Pas de ticket : suis la procédure de <dossier du skill>/references/audit-rtk.md (section Procédure) et réponds en 5 lignes maximum. Dossier du skill : <chemin absolu>."`

Installé en plugin, les fichiers du skill sont remplacés à chaque mise à jour du plugin : les leçons apprises localement sont alors perdues (le journal repart de la version publiée).

## Procédure (pour l'agent)

1. Lance, sans rien d'autre :
   - `rtk discover -a -s 7 -l 8`
   - `rtk session`
   - `rtk learn -a -s 7 --min-occurrences 2`
2. Déduis au plus 5 leçons nouvelles ou corrigées :
   - commande de « MISSED SAVINGS » avec ≥ 10 occurrences → règle « `<commande>` → `<équivalent rtk>` » ;
   - commande de « TOP UNHANDLED » qui sert à lire ou modifier un fichier (`cat`, `python - <<EOF`, `sed -i`, `cat >> fichier`, `echo > fichier`) → règle « utiliser l'outil Read / Edit / Write » ;
   - correction trouvée par `rtk learn` → règle courte (mauvaise commande → bonne commande).
3. Mets à jour la section `## Leçons RTK` de **ces trois fichiers**, avec les mêmes lignes :
   - `<dossier du skill>/../../agents/executant-haiku.md`
   - `<dossier du skill>/../../agents/executant-sonnet.md`
   - `<dossier du skill>/SKILL.md`
   Une ligne par leçon, **10 lignes maximum** par section : fusionne les doublons, remplace la leçon la moins fréquente si besoin. Ne touche à rien d'autre dans ces fichiers.
4. Ajoute une ligne en haut du journal ci-dessous : date, adoption moyenne (`rtk session`), tokens récupérables (`rtk discover`), leçons ajoutées/modifiées. Garde au plus 10 lignes de journal.
5. Réponse finale (5 lignes max) : adoption, tokens récupérables, leçons modifiées.

## Journal (plus récent en premier)

| Date | Adoption | Récupérable | Leçons |
|---|---|---|---|
| 2026-11-17 | — | — | Leçons initiales livrées avec le plugin |
