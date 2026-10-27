# PNJ immobiles et sons d'interaction

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `Character/NPC/Plant/Odar/Odar_01` (arbre géant), act `Act/Player/04_Normal/ArtifactInteraction*` (gravure à un artefact de téléportation)
- **Méthode** : export des séquences en glTF et lecture des clés par os ; lecture JSON du blueprint d'animation et des acts (CUE4Parse) ; médias Wwise décodés et comparés (durée, enveloppe d'amplitude)

## Établi
### Odar_01 : un arbre qui ne bouge pas par ses séquences
- `NPC_Odar_01_Idle` et `NPC_Odar_01_Sidle01_Loop` (21,33 s, 641 clés à 30 i/s) : **aucun** canal ne varie (poses figées).
- `NPC_Odar_01_ConstantLoop` : une seule clé (la deuxième) s'écarte de 9,4° sur la tête et 17 os de branches ; toutes les autres sont
  identiques à la première. Ce n'est pas un balancement.
- `ABP_NPC_Odar_01_PP` (blueprint de post-traitement) contient : lecteur de séquence (`ConstantLoop`) → `ApplyAdditive` sur la pose
  d'entrée, plus un `PoseBlendNode` (asset de poses, probablement les yeux ou le visage).
- Le feuillage a ses propres matériaux (`MI_NPC_Odar_01_Leaf`) : la vie visible de l'arbre doit venir du vent de ces matériaux.

### Son de l'interaction de gravure
- L'act `ArtifactInteraction_Sound` poste deux évènements Wwise : `PC_ArtifactInteraction_Loop_1P` / `_3P` au début, et
  `_Loop_1P_Stop` / `_3P_Stop` ; l'act est réglé `bStopSoundOnPlayerEnds` = vrai.
- Le média attaché à la boucle est un échantillon de ≈ 14 s ; les médias trouvés sous les évènements « Stop » ont la **même** durée et la
  **même** enveloppe : il n'y a pas de son de fin distinct, l'évènement « Stop » arrête la boucle (fondu côté Wwise).

## Supposé
- Le `PoseBlendNode` d'Odar anime le regard ou le clignement ; non vérifié.
- Le fondu d'arrêt de la boucle est réglé dans le projet Wwise (non lisible depuis les seuls médias).

## Pour reproduire
1. Exporter les trois séquences d'Odar_01 et comparer, os par os, l'écart angulaire de chaque clé avec la première.
2. Lire le JSON de `ABP_NPC_Odar_01_PP` : types des nœuds d'animation.
3. Décoder les médias des évènements `PC_ArtifactInteraction_*` et comparer leurs durées et leurs RMS par demi-seconde.
