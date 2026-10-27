# Animations du personnage : saut, armes rangées

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `Character/Player/GF/Ani/` (personnage féminin ; le masculin suit la même organisation), squelette `SK_GF_RootMesh`
- **Méthode** : liste des chemins des archives, lecture des séquences (durée, transformation des os par instant) et des sockets du squelette (CUE4Parse)

## Établi
### Saut (exemple : espadon, préfixe `GF_GS_`)
- Cinq séquences : `Run_Jump_Start` (0,80 s), `Run_Jump_Loop_Start` (0,83 s), `Run_Jump_Loop` (1,00 s), `Run_Jump_Land` (1,00 s),
  `Run_Jump_LandRun` (0,93 s).
- `Loop_Start` est la **transition entre la montée et la chute** ; sans elle (départ puis boucle directement), le passage au sommet est
  brusque.
- Dans `Run_Jump_Start`, le bassin (`Bip001`) ne bouge qu'à la première image, puis reste fixe : le geste est porté par les membres.
- Un saut au sol court (vitesse de départ ≈ 6,6 m/s, gravité ≈ 24,5 m/s²) monte en ≈ 0,27 s : bien moins que la durée du départ.
  Le moteur d'AION 2 doit donc accélérer ou raccourcir ces séquences ; non mesuré dans le jeu.

### Armes rangées
- Le squelette porte deux jeux de sockets : `Equip*` (en main) et `UnEquip*` / `Unequip*` (rangée), par famille :
  `UnEquipGSword`, `UnEquip_GS`, `UnEquip_SW`, `UnEquip_MA`, `UnEquip_ST`, `UnEquip_BW` (sur l'os `UnEquip_WP`, dans le dos),
  `UnEquipOrb` / `_Sub` (`UnEquip_OR`), `UnequipMbook` (`UnEquip_MB`), `UnEquipShield` / `UnEquipTShield` (`UnEquip_SH`),
  `UnequipDagger` / `_Sub` (`UnequipDagger_L` / `_R`), `UnEquipMace` (`Bip001`), `UnequipTSword` (`UnEquip_WP`).
- Aucune séquence de dégainage ou de rengainage n'apparaît dans `Character/Player/GF/Ani/` (seules des postures sociales
  `GF_Fly_Social_Weapon_<arme>_Start/Loop/End`). La locomotion hors combat existe en banque `GF_NoWeapon_*`.

## Supposé
- Le passage paix/combat se fait par simple changement de socket (arme en main ↔ arme rangée), éventuellement avec un effet visuel,
  et par changement de banque de locomotion (`NoWeapon` en paix) ; à vérifier dans les blueprints d'animation du personnage.

## Pour reproduire
1. Lister les chemins `Character/Player/GF/Ani/` contenant `Jump`, `Equip`, `NoWeapon`.
2. Lire les sockets de `SK_GF_RootMesh` (nom, os, transformation).
3. Échantillonner `Bip001` et les mains dans `Run_Jump_Start` à quelques instants.

## Mise à jour : prises en main et poses (build 25767555, fin octobre 2026)
- Rengainer : confirmé, aucun geste de rengainage ; l'arme passe sur un socket `UnEquip_*` et la banque `NoWeapon` prend le
  relais.
- Prises en main (`Equip*`) : `EquipGSword`, `EquipTSword`, `EquipTShield`, `EquipShield` (os `ShieldWeapon`),
  `EquipDagger` / `_Sub` (`DaggerWeapon` / `_Sub`), `EquipOrb` / `_Sub`, `EquipStaff` / `_Sub`, `EquipMace` (os
  `MaceWeapon`, enfant de la main droite), `EquipMbook` (os `MbookWeapon`).
- Côté homme, les **99 prises** sont portées par le squelette `SK_GM_Root`, pas par le maillage `GM_RootMesh` ; côté
  femme, elles sont sur le maillage.
- Bouclier (`ShieldWeapon`) : GF translation (11,118 ; −0,511 ; 8,600) cm, quaternion (0,4978 ; −0,5188 ; −0,5074 ;
  0,4750), échelle **0,85** ; GM (13,061 ; 0 ; 11,181), quaternion (−0,5 ; 0,5 ; 0,5 ; −0,5), échelle 1. Dans les gestes,
  l'os `ShieldWeapon` reste à l'origine : le bouclier se fixe avec la pose de référence de l'os.
- Grimoire : squelette `MB_Skeleton`, `EquipMbook` à l'échelle 0,8 (GF) et 1 (GM), repos `UnequipMbook` (os `UnEquip_MB`).
  Le livre n'a pas d'animation propre : pose de référence ouverte en V, charnières `Bone_MB_1/2` ; ce sont les animations
  du personnage qui déplacent `MbookWeapon`. `EquipOrb_Sub` est aussi sur `MbookWeapon`.
- Os d'arme (`MaceWeapon`, `ShieldWeapon`…) : ils ne bougent qu'avec les animations qui les portent.
- Pieds au repos sur la pointe (talon 9 cm au-dessus de la plante). Pose `GF_Flatshoes_Pose` pour les chaussures plates :
  pied −35° autour de Z local, orteil +30° et décalé de (−0,097 ; 2,633 ; 0) cm, `Bip001` abaissé de 6,7 cm.
- Combat (montages, esquives, banques) : voir [animations-combat](animations-combat.md).
