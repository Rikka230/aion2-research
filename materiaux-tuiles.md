# Matériaux : tuiles répétées (EM_Object_Tile / EMM_Object)

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : troncs d'arbres communs, ex. `BG/Common/Nature/Tree/TreeE/COM_TreeE02a_MI` (maître `EM_Object_Tile`, lui-même sur `EMM_Object`)
- **Méthode** : lecture des propriétés de l'instance de matériau et de la texture (CUE4Parse, sortie JSON) ; reproduction dans Unreal et comparaison en capture

## Établi
- La couleur du tronc n'est pas dans `T_BaseColor` (texture neutre blanche) mais dans `T_Tile_BaseColor`, avec sa normale dans
  `T_Tile_Normal_DXT1` et son masque dans `T_Tile_ARM` (interrupteur `Use_Tile_ARM` actif).
- Ces textures de tuile sont petites : `COM_TreeE02a_D` fait **512 × 512** dans le jeu (ImportedSize 512, 10 niveaux de mip, aucun
  niveau supérieur dans les données en flux).
- Le paramètre scalaire `Tile_Base_Scale` donne la répétition : **12** pour `COM_TreeE02a_MI`, **15** pour le tronc de `COM_TreeE01`.
  La tuile est répétée ce nombre de fois sur les UV du maillage ; étirée une seule fois, elle paraît floue et pixelisée de près
  (grandes taches au lieu d'un grain d'écorce).
- L'instance ajoute une normale de détail générique (`T_Normal_DXT1` = une normale d'écorce de `BG/Common/TileTexture/Bark/`,
  `Detail Normal Scale` 6, `Detail Normal Intensity` 2).
- À l'inverse, les écorces des arbres de `Tree_ENV` (ex. `DTT_AlderA01_MI`, maître `EMM_Tree_Trunk`) ont `Base_Scale` = 1 : leur
  texture « DTT » est déjà répétée par les UV du maillage.

## Supposé
- Tous les objets sur `EM_Object_Tile` suivent la même règle (UV × `Tile_Base_Scale`) ; seuls deux arbres ont été vérifiés.
- La normale de détail est appliquée à l'échelle `Detail Normal Scale` par-dessus la tuile ; non mesuré.

## Pour reproduire
1. Lire l'instance de matériau du tronc : relever `T_Tile_*` et `Tile_Base_Scale`.
2. Dans un matériau de reproduction, multiplier les coordonnées UV par `Tile_Base_Scale` avant d'échantillonner les trois textures de tuile.
3. Comparer de près avec et sans la répétition.
