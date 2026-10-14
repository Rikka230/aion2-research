# Matériau de paysage (EM_Landscape)

- **Build du client** : Steam 25767555, octobre 2026
- **Sujet** : `/Game/Material/BG/EM_Landscape` et ses instances `EMI_*` (dossiers `LandscapeMI` des cartes partagées)
- **Méthode** : export JSON des instances avec CUE4Parse (paramètres scalaires, couleurs, textures, interrupteurs).

## Établi

- Huit couches : `Cliff`, `Grass`, `Cover`, `Road`, `Side`, `Dirt`, `Mix`, `Wet`. Les matériaux physiques portent les
  mêmes noms (`BG/LandscapePhysical/<Couche>_Layer_PhysicalMaterial`, avec une variante `_Weather_`).
- Chaque couche a deux échelles de répétition, près et loin, fondues selon la distance :
  `<Couche>_Tiling_Near` / `_Tiling_Far` (ou `_Tiling` / `_Tiling_Far`), et `<Couche>_FarNearPoint`,
  `_Blend_Distance` ou `Cliff_DistanceBlend`. Exemple des Highlands : falaise 0,8 de près, 0,5 de loin, fondu à 130 m.
- Échantillonnage « stochastique » contre la répétition visible : `<Couche>_Stochastic_Scale` et `_Contrast`.
- Normales : interrupteurs `Use_<Couche>_Dxt1Normal` ; la vraie normale est alors `<Couche>_Normal_DXT1`.
- Réglages de couleur par couche : `_Brightness`, `_Saturate`, `_Color_Tint`. La falaise des Highlands prend une roche
  peinte assez lisse en couleur (`DTT_RockA01_D`) et une normale de falaise détaillée (`TRE_Cliff_A01_N`), intensité 2.
- Les grandes parois ne reposent pas sur le paysage seul : `BG/Common/Nature/Mountain` (`COM_MountainA01` à `G02`) et
  `BG/Common/Nature/Stone/Cliff` (plusieurs dizaines de falaises) sont des maillages posés par-dessus.

## Supposé

- L'unité de `Tiling` n'est pas encore établie (probablement un facteur appliqué à une échelle de base du maître, dont
  le graphe n'est pas lisible dans les archives cuites).

## Pour reproduire

1. Lister les `LandscapeMI` dans l'index des archives.
2. Exporter une instance en JSON et lire `ScalarParameterValues`, `TextureParameterValues` et `StaticParameters`.
