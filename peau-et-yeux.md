# Peau, maquillage, sourcils, yeux

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : maîtres `CMM_Head`, `CM_Head_Cstm`, `CM_Head_Glitter`, `CMM_Body`, `CMM_EyeAO` ; textures
  `Character/Player/Common/CustomMI/*`, `GF_Head_D`, `Cstm_EyeBrow_T2A`, `Cstm_Pupil_01`, `T2A_BodyType`
- **Méthode** : shaders compilés désassemblés (voir [shaders-compiles](shaders-compiles.md)), lecture des canaux des
  textures, paramètres des instances (CUE4Parse) ; reproduction dans Unreal 5 et comparaison en capture

## Établi

### Visage (`CMM_Head`)
- La diffuse de tête est commune à toutes les têtes (`Player/Common/GF_Head_D`) ; chaque tête n'a que sa normale.
- Teinte : `D × lerp(Skin_Color, 1, 0,75 × masque.A)`. Canal B du masque ARSC = diffusion (opacité de diffusion bornée
  entre 0,11 et 1) ; spéculaire constante 0,7 ; rugosité plancher 0,2.
- Maquillage : huit couches (ombrage, illuminateur, blush, deux fards, eye-liner, deux couches de lèvres), une texture
  par couche (`MK_Head_*`), paramètres `<Type>Color` et `<Type>Alpha`. Sous le maquillage, la rugosité change :
  ≈ 1,0 sous l'eye-liner, 0,418 et 0,35 sous les deux couches de lèvres.
- Canaux des décalques : `MK_Head_EEE_xx` (œil) R = fard, G = liner, B = coin ; `MK_Head_LFL_xx` (lèvres) R ;
  `MK_Head_HBS_xx` joues.
- 77 instances `CustomMI` pour 8 types de calques. Il faut toujours fusionner l'instance parente de la famille
  (`Xxx_00`), qui porte les réglages de base.

### Calques (`CM_Head_Cstm`)
- Paramètres : taches de rousseur (`FreckleAlpha`), grains de beauté (`MoleLeftRight`…, décalage, échelle, rotation),
  tatouage (centre, biais, canal, `TattooColor`), barbe naissante et rasée, cicatrice (couleur, texture, rotation,
  `ScarDepth` → `BurntArea_NormalStrength`, relief `Burnt_NmR`), peinture de guerre, sourcils (`EyebrowDist`,
  `EyebrowUpDown`, `EyebrowRotate`, `EyebrowWidth`, `EyebrowThickness`, `EyebrowHairIntensity`, `EyebrowRoughness`).
- Cicatrice : UV = rotation(p) / échelle + 0,5. Le préréglage met `Burnt_UseColor` à 0,2 (teinte à 20 %).
- Les textures de masque doivent être lues en linéaire.

### Sourcils
- `Cstm_EyeBrow_T2A` : B = crayon, A = poils. `EyeBrow_Roughness` 0,45 ; métal = 0,55 × poils ; normale `NMPH.xy`
  (x inversé du côté en miroir) × `EyeBrowNormal_Intensity` × saturate(crayon + poils), combinée par
  `BlendAngleCorrectedNormals` ; diffusion 0,3 sous crayon et poils.
- La couche des sourcils (`EyeBrow_NUMBER`) sert de masque à l'échelle du visage entier : si elle manque, le maître lit
  l'alpha (= 1) de la texture noire par défaut et tout le visage prend la couleur et le métal 0,55 des sourcils.

### Paillettes
- `CM_Head_Glitter`, fonction `CMF_Cstm_Glitter`, masque `T_FlakeMaskWhite`. Les zones `GlitterArea` multiplient les
  masques du blush, des deux fards et de la première couche de lèvres.
- Une porte `saturate(GlitterUse × round(zone))` décide de l'éclat : avec une zone à 0,1, aucune paillette.
- Effet sur la couleur, la rugosité (`Glitter_RoughMin`), le métal, le spéculaire (`Glitter_Specular`). Valeurs par
  défaut dans `CachedExpressionData` de `CMM_Head`.

### Corps (`CMM_Body`)
- `Skin_Color` = 0,57. Relief des muscles : couche 1 de `T2A_BodyType`, pondérée par `Mask_BodyParts`, combinée en RNM ;
  `Leg_Intensity` règle les jambes. Veines : couche 2 × `NormalVein_Multipy` (orthographe d'origine).
- Tatouages du corps : six zones (UV1 pour le torse, UV2 pour les membres), textures sRGB en bord bloqué, interrupteur
  `<zone>_Use`. La texture par défaut est noire et opaque : sans l'interrupteur, elle noircit les membres.

### Œil
- Modèle d'ombrage Eye, profil de diffusion `Human_Eye_SSS`. Masques d'iris, renflement, veines ; réfraction de la
  cornée ; profondeur `T_EyeMidPlaneDisplacement` en 16 bits. Spéculaire 0 ; IrisMask et IrisDistance en CustomData0/1.
- Globe à UV planaires de face, apex de la cornée en (0,5 ; 0,5). Iris `Iris_02_M` : disque de rayon 0,48 de l'image,
  B = dégradé radial. Sclère `T_Sclera_000_SV`.
- Pupille `Cstm_Pupil_01` : l'alpha vaut 255 partout, le disque de la pupille est dans **R**.
- `MI_Eye_AO_01` n'est pas un film d'ombre : ce sont les cils (`Eye_AO_M_01`, G haut, B bas, seuil 0,333).
  `MI_EyeAO_Tear` est la ligne de larme (`Eye_AO_M_07`, R).
- Reflet : `EyeHighLightTexOffset` = (1, 1, 0,024, 1) ; tronqué à trois composantes, le reflet sort de l'œil.

### Reproduction dans Unreal 5 (vérifiée)
- Modèle Eye : la normale du plan de l'iris se branche sur le nœud **Tangent Output**, pas sur la broche Tangent ;
  sinon l'iris reste presque noir.
- Le visage et le corps reproduits avec `Skin_Color` et la formule ci-dessus perdent le défaut « peau trop claire » que
  donne une diffuse prise telle quelle.

## Supposé
- La rugosité de l'œil (lecture « 0,1 → 0,07 ») est ambiguë : interpolation ou correction.
- La parallaxe de l'iris existe mais son effet est faible aux valeurs d'AION 2 ; non reproduite.

## Pour reproduire
1. Désassembler le pixel shader de `CMM_Head` et suivre les sorties couleur, rugosité et métal.
2. Afficher les canaux de `MK_Head_*`, `Cstm_EyeBrow_T2A` et `Cstm_Pupil_01` séparément.
3. Tester un visage sans couche de sourcils : le visage entier doit noircir.
