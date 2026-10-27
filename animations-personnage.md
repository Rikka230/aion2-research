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
