# Armures, tenues et armes : tables, nommage, matériau CM_Armor

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `Character/Player/G?/Armor/<série>/`, `Character/Player/Weapon`, tables `UnrealData/ArmorCostumeResourceTable`,
  `WeaponCostumeResourceTable`, `PcBasedNPCResourceTable`, `PcResourceTable` ; maître `CM_Armor`
- **Méthode** : export JSON des tables et des instances, export glTF des pièces, shader compilé de `CM_Armor` (voir
  [shaders-compiles](shaders-compiles.md)) ; reproduction dans Unreal 5 comparée aux icônes d'objets

## Établi

### Nommage
- Armures : `G?_<série>_T<palier>_<pièce>`, pièces `Body`, `Pants`, `Boots`, `Glove`, `Shoulder`, `Cape`, casque.
  Exemple : `Character/Player/GF/Armor/0005/GF_0005_T05/GF_0005_T05_Body`. Costumes nommés dans `Armor/Special/`
  (ex. `GF_Kromede_01`). La casse des pièces varie (`GM_0041_T04_pants`).
- Armes : `<famille>_<série>_T<palier>`. Familles : `GS` espadon, `SW` épée, `SH` bouclier, `DA` dague, `BW` arc,
  `MB` grimoire, `OR` orbe (`_R` / `_L`), `MA` masse, `ST` bâton. Pas d'arme à feu.
- Une seule série « 0000 » : la tenue de départ. Ligne `GF_0000_T01` → maillages `GF_0000_T01_*` (7 pièces) ; ligne
  `GM_0000_T01` → maillages **`GM_0133_T01_*`**. Les icônes `Icon_Equip_AR_L_0133_T01_*` en font un objet d'équipement.
- Tenue « par défaut » : lignes `G?_Light_Default` et `G?_Dark_Default` → `G?_BaseBody`, `G?_BasePants`,
  `G?_BaseBoots`, identiques pour les deux races. Sous-vêtements : `Armor/Special/G?_Underwear_01` (`_Body`, `_Pants`).
- Les ensembles examinés n'ont qu'un matériau `_LT`, sans `_DK` : même tenue pour les deux races.
- Neuf séries citées par la table de costumes manquent dans les archives (0073, 0172, 0198, 0204, 0216, 0076, 0082, 0087,
  0185). `WeaponCostumeResourceTable` ne relie pas une arme à une tenue.
- Ordres de grandeur : 157 séries féminines T01 à T06 et 42 costumes nommés ; bas niveau T01 (21 armes, 19 séries
  d'armure) et T02 (38 armes, 24 séries).
- Certains corps de tenue ont leur propre squelette, à importer à part (`GM_0049_T05_Body`, `GM_0071_T06_Body`,
  `GM_0032_T05_Body` ; ce dernier demande 12 os absents de `GM_RootMesh_Skeleton`).
- `PcBasedNPCResourceTable` décrit des PNJ de test vêtus d'ensembles cohérents par classe (ex. `NPC_TEST_F_Gladiator` →
  `GF_0005_T05`, `NPC_TEST_M_Chanter` → `GM_0008_T05` avec le casque `GM_0009_T05`).

### Matériau `CM_Armor`
- Couleur = lerp(D', saturate(Σ saturate(lum(D') + `CustomMask<c>_Value` − 1) × `CustomMask<c>_Color` × m_c),
  `Use_ColorCustom` × Σ m_c), où m_c sont les canaux du masque `CSTM`.
- Rugosité, métal, AO et normale de détail sont réglés par canal (`*_MatType`, `*_GradationUse`, `CustomMask*_Use`).
  Textures de détail répétées : `T_DetailNMA_06_Leather`, `T_Detail_MetalA03_NMMR`, `T_Detail_ClothA03_NMMR`,
  `T_Detail_CottonA01_NMMR`.
- Masque `SE` : R = diffusion, G = émissif.
- Jeu de textures d'une pièce : `_D`, `_CSTM`, `_N`, `_ARM` (ex. `GF_0039_T05_Body_*`).
- Les couleurs `CustomMask` par défaut des instances reproduisent déjà les icônes d'objets (écart 0,045 sur un champagne).
  Certains ors sont des métaux champagne (métal 0,97 à 1) dont l'aspect vient surtout des reflets.
- Les armes ont un maître distinct. Leur maillage déclenche un repli de matériau incompatible avec Nanite à l'import.

### Prises en main
Voir [animations-personnage](animations-personnage.md) (sockets `Equip*`, bouclier, grimoire).

## Supposé
- Sens des canaux `CSTM` (R cuir, G métal, B tissu teint, A tissu clair) : déduit des formules et de rendus, pas relevé
  canal par canal sur toutes les pièces.
- `CustomMask*_Use` (valeurs de 1 à 5) désignerait des emplacements de teinture remplis à l'exécution.
- Le doré plus chaud de l'écran officiel viendrait de l'éclairage ou d'une teinture appliquée au lancement ; non tranché.

## Pour reproduire
1. Exporter `ArmorCostumeResourceTable` et suivre une ligne jusqu'à ses maillages et instances.
2. Lire `CustomMask*_Color` / `_Value` dans l'instance et appliquer la formule ci-dessus dans un matériau Unreal 5.
3. Comparer le rendu sous un éclairage neutre à l'icône de l'objet.
