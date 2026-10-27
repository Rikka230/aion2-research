# Physique des cheveux, vêtements et ailes : KawaiiPhysics

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : AnimBP `ABP_*_Hair_*`, `ABP_G?_Cape_3Line_Short`, `ABP_GF_0032_T05_Body`, `ABP_WingA_001` ;
  structures `Character/Player/BP/Struc/KawaiiPhysics_Breast`, `_Acc_Heavy`, `_Acc_Light`, `_Acc_String` ; courbes
  `Hair*Curve*` ; corps physiques `*_PA` et `*_Physics`
- **Méthode** : export JSON des AnimBP (nœuds d'animation, liaisons PropertyAccess), relevé des chaînes d'os ;
  reproduction dans Unreal 5 avec le plugin libre KawaiiPhysics (licence MIT)

## Établi

### KawaiiPhysics dans AION 2
- AION 2 utilise KawaiiPhysics pour les cheveux, la poitrine, les accessoires, les capes, les jupes et les plumes des ailes.
- **322 AnimBP** contiennent **2 151 nœuds** `AnimGraphNode_KawaiiPhysics*`.
- Les `PhysicsSettings` visibles dans l'objet par défaut sont des valeurs de remplissage. Les vrais réglages sont liés par
  **PropertyAccess** à des variables (`LongSide`, `LongFront`, `MiddleBackUnder`…), remplies au démarrage depuis une
  structure de réglages des cheveux (nommée « Hair Physcis », entrées `LongSide_<n>_<GUID>`) : **2 556 réglages liés**.
  Exemple, mèches latérales : amortissement 0,3, raideur 0,24, angle limite 20°. Certaines mèches libres ont une
  raideur de 0,05.
- Courbes : `HairDampingCurve_01`, `HairStiffnessCurve_0*`, `HairRadiusCurve_0*`, `HairLimtAngleCurve_0*`
  (orthographe d'origine), `HairWindRateCurve_Long/Short_01`, `WindNoise*`. Collisions : `kawaiicollision_body`.
- La gravité est une force calculée en **code natif** (`NativeEquipSimulation.ExternalForce_L/R`), absente des assets.
- `CR_Hair` (Control Rig) replace les mèches d'épaule avant la simulation.
- La racine de chaque mèche reste animée.

### Vêtements
- 1 445 assets de corps physiques `*_PA` (`GF_Skirt_L/R_PA`, `GF_Common_Dress_L/R_PA`, y compris en pose montée) et
  quelques `*_Physics` (casques, corps).
- Chaînes d'os : `L/C/R_Cape_nn` sous des racines `Cape_*_Root` ; `Skirt_*` sous `Skirt_Root` ; manches `LinkElbow_*`.
- Une pièce à physique tire son AnimBP de la **table des costumes** (colonne `AnimClass` de
  `ArmorCostumeResourceTable`), rarement de son nom : la cape de départ n'a pas d'ABP à son nom. 521 maillages ont une
  animation attitrée.
- `ABP_GF_0032_T05_Body` porte des capsules au bassin, aux cuisses et aux mollets : la jambe repousse la jupe.
- Cape masculine de départ (`GM_0133_T01`) : liée pendante (`C_Cape` de 1,61 à 1,12 m), mais le squelette du corps porte
  les os de cape à l'horizontale. Il faut remettre ces os à la pose de liaison de la cape avant la simulation.

### Ailes
- `ABP_WingA_001` : KawaiiPhysics des plumes, 12 nœuds dans `WingA_PP`. Voir [ailes-et-vol](ailes-et-vol.md).

### Reproduction dans Unreal 5 (vérifiée)
- KawaiiPhysics 1.21.0 compile sous Unreal 5 récent et accepte les réglages relevés tels quels.
- `SimpleExternalForce` est un déplacement par seconde ajouté à chaque pas, pas une accélération. Une gravité de 980 cm/s²
  vers le bas avec `bUseLegacyGravity` remplace correctement la force native.
- Il faut recalculer les références d'os quand un niveau de détail en retire, sinon plantage (`Pose.IsValidIndex`).
- KawaiiPhysics met à l'échelle le décalage des collisions, pas leurs rayons ni leurs longueurs : sur un personnage
  réduit par la morphologie (échelle de `Root`), multiplier rayons et longueurs par cette échelle.

## Supposé
- La valeur de la force native (vent, inertie de déplacement) n'est pas connue ; seule son existence l'est.
- Les manches `LinkElbow_*` passeraient par une couche d'animation partagée, non retrouvée.
- L'attache des cheveux (bouton `HairTied`) passerait par l'AnimBP des cheveux.

## Pour reproduire
1. Exporter une `ABP_GF_Hair_*` en JSON et compter les nœuds KawaiiPhysics.
2. Pour un nœud, suivre `AnimBlueprintExtension_PropertyAccess.Library` jusqu'à la variable, puis jusqu'à l'entrée de la
   structure de réglages.
3. Comparer ces valeurs à celles de `PhysicsSettings` dans l'objet par défaut : elles diffèrent.
