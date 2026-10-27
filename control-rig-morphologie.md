# Control Rig cuites : morphologie du corps et du visage

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `CR_GF_CstmBody`, `CR_GM_CstmBody` (morphologie), AnimBP `ABP_PC_GF_BodyCustomize` /
  `ABP_PC_GM_BodyCustomize`, table `PcCustomizeFeatureTable_Bone`, Pose Asset `GF_Head_Cstm_PA` ; autres rigs
  `CR_Hair`, `CR_Seq_Cape`, `CR_Seq_Hair`, `CR_Seq_Skirt`, `CR_GF_FaceRig`
- **Méthode** : lecture JSON des Control Rig (CUE4Parse), décompilation du bytecode RigVM en listing, exécution du
  programme par un interpréteur, comparaison avec les curseurs du jeu

## Établi

### Ce qui reste d'une Control Rig cuite
- Le graphe d'édition n'est pas dans le jeu. Il reste **tout le programme RigVM** : bytecode, mémoire des littéraux,
  chemins d'accès aux os. Ce programme suffit à exécuter la rig : inutile de deviner les formules.
- CUE4Parse échoue sur `GraphFunctionStore` et sur la `RigHierarchy` de ces rigs ; le JSON partiel n'est pas exécutable
  tel quel. Il faut décoder soi-même le bytecode et la mémoire.
- Un interpréteur qui couvre **29 fonctions RigVM** et propage les transformations dans la hiérarchie (os virtuels
  compris) suffit pour la morphologie du corps.
- AION 2 contient **deux versions** des rigs de corps : une ancienne rig complète, qui calcule tout en RigVM, et la rig
  « native » actuelle, qui délègue une partie des calculs à du **code natif absent des archives**. Quand les deux
  existent, la version non native est la seule reproductible entièrement.

### Morphologie du corps
- `PcCustomizeFeatureTable_Bone` : **42 réglages de corps** et **59 de visage** par genre, bornes Min / Mid / Max
  (plages parfois inversées).
- La taille passe par l'échelle de l'os `Root`. Les os de silhouette : `CstmBMI`, `ButtScaleReset`, `ThighThickReset`,
  `Pelvis_Width`, `L/R_Butt`, `L/R_Quadriceps`, `CalfMuscle`, `Stomach`, `Deltoid` ; poitrine par l'échelle de
  `b_L/R_Breast_Base`. Ces os ont peu d'influence chacun (`L_Butt` n'a pas de poids direct).
- La rig actuelle ajoute biceps et triceps féminins, bornés de **0,8 à 1,2** (échelle d'os).
- La corpulence (`BMI`) passe par des décalages pondérés par le BMI ; le dos, le ventre et l'échelle du bassin sont
  recalculés en code natif. Appliquer les décalages bruts donne un ventre démesuré.
- Le programme ne lit la pose de référence que pour l'os d'effet d'arme, pas pour les membres : il s'exécute sur la pose
  animée courante.

### Visage
- Les curseurs du visage pilotent **184 poses additives** (`GF_Head_Cstm_PA`), pas une rig.
- Animation faciale : les animations portent des courbes de type ARKit (`eyeBlinkLeft`, `jawOpen`, `mouthSmileLeft`…)
  et des clés `Mph_*`. Un Pose Asset FACS par tête (ex. `Head/GF_Head_023/GF_Head_023_FACS_PA`) traduit chaque courbe en
  pose additive : 52 poses de 30 à 130 os. Les additifs s'appliquent sur la **pose de liaison du maillage de tête**, pas
  sur le squelette générique `SK_GF_Head` (écart de l'ordre de 2 cm : chaque tête place ses os à sa forme).
- Les clés `Mph_*` ne touchent que la bouche.

### Autres rigs repérées
- `CR_Hair` replace les mèches d'épaule avant la simulation physique (suivi de la clavicule, état de déplacement).
- `CR_Seq_Cape`, `CR_Seq_Hair`, `CR_Seq_Skirt` servent aux séquences ; `CR_GF_FaceRig` aux cinématiques.

## Supposé
- La valeur native du ventre en corpulence reste inconnue ; seule son existence est établie.
- `Character/Player/BP/FacialRigCurve` servirait aux cinématiques ; non relu.

## Pour reproduire
1. Exporter la Control Rig en JSON et isoler le bytecode, la table des opérandes et la mémoire des littéraux.
2. Écrire un listing lisible (une ligne par instruction, opérandes résolus), puis un interpréteur des fonctions présentes.
3. Exécuter le programme une fois par image après l'animation, sans cumul, et comparer aux bornes Min / Max du jeu.
