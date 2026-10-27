# Ailes et vol

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `WingA_001` (`M_WingA_001`, `T_WingA_001_DO`), squelette `WingA_Skeleton`, `ASL_WingA`,
  `Character/Player/GM/Ani/Normal/Fly/WingA/*`, `ABP_WingA_001`
- **Méthode** : liste des chemins, lecture des séquences et de l'AnimBP (CUE4Parse), lecture des canaux du masque `_CSTM`

## Établi
- Les ailes sont un maillage squelettique à part, avec leur squelette `WingA_Skeleton`, accroché à l'os `Wing_Root` du
  personnage.
- Séquences d'ailes (15) : envol `WingA_Fly_Start`, vol sur place `Fly_Idle`, vol en avant `Fly_Forward`, montée,
  descente `Fly_Down`, départ du planage, planage, atterrissage. Hors du vol, les ailes sont repliées (`WingA_Hidden`).
- Côté personnage (Gladiatrice, espadon) : 13 animations de vol — envol, attente, quatre directions, montée, descente,
  atterrissage, planage. Esquives en vol : voir [animations-combat](animations-combat.md).
- `ABP_WingA_001` : simulation KawaiiPhysics des plumes, 12 nœuds dans `WingA_PP`.
- Masque `_CSTM` de `WingA_001` : R = beige (`BCB293`), G = gris (`A7A7A7`) par défaut.
- Catalogue d'apparences : 36 ailes. Les émotes ont des variantes d'ailes `WingA` à `WingF`
  (`SocialAction/WingA/AM/WingA_Social_Wing_AM`).
- Caméra : en vol et en planage, le champ de vision gagne 10° ; sur un courant aérien (Windpath), 40°. Voir
  [controles-et-camera](controles-et-camera.md).
- Actions d'entrée : `Move_Fly`, `Move_FlyUp`, `Move_FlyDown`, `Glide_Boost`, `Glide_DashUp`, `Glide_Rotate`.
- La montée de niveau n'utilise **aucun** maillage ni texture d'aile (recherche dans `FX/Act/UI`, `Common_LevelUp`,
  `NS_UI_Levelup_2`).

## Supposé
- Le matériau de vol complet (`M_WingA_001` et ses effets) n'a pas été relu dans le shader compilé ; une reproduction à
  partir de la seule texture `_DO` paraît grise et plate.
- Valeurs de planage (gravité, vitesse de chute, durée de la jauge) : non trouvées dans les données lisibles.

## Pour reproduire
1. Lister `WingA` dans l'index et exporter `ABP_WingA_001`.
2. Lire les séquences de `GM/Ani/Normal/Fly/WingA/` et leur squelette.
