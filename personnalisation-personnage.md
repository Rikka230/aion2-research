# Création et personnalisation du personnage : tables, réglages, caméra

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : tables `PcCustomizeFeatureTable_*`, `CustomizingUIListTable`, `MaterialCustomizeParamSettings`,
  `PCCustomizeCommonData`, `PreviewSceneCameraPresetTable_Customizing` ; fenêtre `UI/Widget/FWindow/FWindow_Customizing`
- **Méthode** : export JSON des tables et des widgets (CUE4Parse), croisement avec les maillages de tête (morph targets) et
  avec les shaders compilés ; comparaison avec des vidéos publiques du client pour le comportement

## Établi

### Tables
| Table | Contenu |
|---|---|
| `PcCustomizeFeatureTable_Bone` | 42 réglages de corps et 59 de visage par genre, bornes Min / Mid / Max |
| `PcCustomizeFeatureTable_Material` | catalogues de couleurs, textures et calques |
| `_PresetFace` / `_PresetBody` / `_PresetMakeup` | 96 / 18 / 35 préréglages |
| `_SubPreset`, `_SubBeardPreset` | couleur, position, échelle, rotation, symétrie, miroir de chaque calque |
| `_InitalizeData` (orthographe d'origine) | valeurs de départ par genre |
| `MaterialCustomizeParamSettings` | 25 liaisons réglage → paramètre de matériau |
| `PCCustomizeCommonData` | voix d'aperçu, liste `ExcludeMakeupSaveData` |
| `CustomizingUIListTable` | type de chaque réglage (`ValueType`), variables `CompVariable` A/B, `ColorVariable`, paires des pavés en croix |

- Préréglages de visage : 46 féminins et 44 masculins standards, plus 6 de collaboration. L'ordre des identifiants n'est
  pas l'ordre d'affichage du catalogue.
- Catalogues (femme / homme) : 38/35 visages, 95/91 coiffures, 100 teintes de peau, 20 pupilles, 5 iris, 10 cils,
  12/11 sourcils, 15 eye-liners, 20 tatouages de visage, 10 cicatrices, 20 peintures de guerre, tatouages du corps
  (zones × 18 motifs), 13 barbes, 7 barbes naissantes. Les catalogues de l'œil sont sans suffixe `_L` / `_R` alors que les
  grilles demandent `PupilShape_L`, `IrisShape_L`… (mode symétrique ou asymétrique).
- Les textures numérotées `1001` / `2001` ne sont **pas** des UDIM (désactiver la détection UDIM à l'import).

### Noms réels des réglages
- Le nom d'un bloc de la page ne suffit pas : `CustomizingUIListTable` donne le vrai réglage. Exemples :
  `Ear_ShapeChange` → `EarShapeChange` ; `bUseBodyGloss` → `bUseBodyGlossiness` ; `EyelashColor` →
  `EyelashColor_Upper` et `EyelashColor_Lower`.
- Deux corpulences : `BMI` et `Thick_AllBody` (épaisseur du corps) ; taille : `Height`.
- Oreilles pointues : morph target `EarShapeChange` des têtes (304 sommets, 2,3 cm au plus).
- Cils : section `Eye_AO` de la tête, morphs `Eye_AO_UpLong`, `Eye_AO_DownLong`, `Eye_AO_DownShort`.
- Brillance de la peau : `FaceGlossiness` et `BodyGlossiness` agissent sur la rugosité (`Roughness_Add`,
  `Body_Roughness_Add`, de −0,35 à 0,35) ; le corps n'est réglé à part que si `bUseBodyGlossiness` est vrai.
- Maquillage : les intensités (`ShadingAlpha`, `BlusherAlpha`, `EyeshadowAlpha`, `LipAlpha`…) attendent une valeur de
  0 à 1. Une instance de calque peut porter plusieurs `*_NUMBER` (ex. `LipA_02` : `EyeLiner_NUMBER` 0 puis
  `LipA_NUMBER` 1) : lire le numéro propre à chaque type. La couche 0 est le choix « aucun ».
- Les préréglages de maquillage appliquent `ExcludeMakeupSaveData` : peau, yeux, sourcils, cils, cicatrices et
  tatouages ne sont pas touchés.
- Tatouages : onglet de type `TattooMultipleSelect` (zones à icônes `UT_FWindow_Customizing_Icon_Parts_*`, 18 motifs par
  zone). Couleur, position, échelle, rotation, miroirs (`bTattooFlipU/V` → `Tattoo_FlipU/V`) et symétrie
  (`bTattooSymmetry` → `Tattoo_Symmetry`) ne valent que pour le tatouage du visage.
- Cheveux deux tons : pas de double curseur mais `RootTipRange` (−1 à 1) et `RootTipContrast` (0 à 20), visibles
  seulement en mode deux tons (`bUseHairColorDetailSettings`). Avec `bUseHairColorDetailSettings` = 0, racine et
  pointe ont la même couleur.

### Types de curseurs
- `OptionSlider` (156 curseurs), `OptionColorSlider` (5), `OptionBorderSlider` (pavé à deux axes, point libre,
  ex. `Eye_Position_BorderSlider`).
- `MinMax` : Mid au centre. `MidMax` : linéaire de Min à Max, un curseur jamais touché est à la position de Mid.
  `MaxMin` : sens inversé (plus d'éclat = moins de rugosité).
- `CustomizingUIListTable` compte **20 pavés** (deux variables chacun, `BorderSliderVariableName_X` / `_Y`) : 9 sur les
  poses du visage, 1 sur la poitrine, 10 sur les calques (sourcils, grain de beauté, tatouage, cicatrice, paillettes).
  La page en contient 22 instances.
- Le réglage de position de la lumière (`P_LightPosition`, `BorderSlider`) n'a pas de `ValueType`.

### Caméra et éclairage de l'aperçu
- `PreviewSceneCameraPresetTable_Customizing` : **15 préréglages**, un par onglet (Preset, FaceDetail, HairDetail,
  MakeUpDetail, BodyDetail, Arms, Torso, jambes via `BodyTattooType_Leg_L`, Waist centré au bassin). Chacun donne la
  distance par défaut et les bornes du zoom. Le point visé glisse de l'os du plan rapproché (ex. la tête) à `Root` +
  100 cm selon la distance. Champ de 40°.
- Studio : `UnrealData/Environment/Setting/Preview_Customizing01` (et `_01_Outdoor`, `02`). Un bloc `CharLightsSetting`
  place trois lumières (principale, appoint, contre-jour) comme un bras de caméra : décalages de rotation, longueur,
  intensité, taille, volets, couleur. Ambiance : AmbientCubemap `approaching_storm_4k`, SkyLight `02_HDR`.
  Voir [eclairage-personnages](eclairage-personnages.md).

### Fenêtres
- `FWindow_Customizing` (pages Visage, Corps, Cheveux, Maquillage), `FWindow_Customizing_OptionColorPicker`,
  `Popup_CustomizingAppearance` (8 emplacements d'apparence), `Lobby_CreateChar`, `P_Customizing` (barbier en jeu).
- Le Blueprint `FWindow_Customizing` ne contient que 6 fonctions (annuler, rétablir, boutique, bandeau) : la logique
  est dans l'exécutable.
- Bouton `UT_Common_Btn_HairTied` : relève les cheveux (il ne les masque pas).
- Sélecteurs de couleur `P_ColorPickerA/B` (`ColorSpectrum`, `ColorSlider`, `Img_Hue`) ; B règle les pointes
  (`Hair2Color`). Palette de teinture des tenues : `DyeColorPaletteTable`, 259 couleurs.
- Sur les écrans de création, `P_Img_BG` reçoit une cible de rendu : sans elle, il masque la scène.

### Tenues d'essai
- Bouton 1 : sous-vêtements (`G?_Underwear_01_Body` et `_Pants`, icône `UT_Customizing_Costume_Basic`), puis trois
  tenues indépendantes de la classe : homme `GM_0108_T04`, `GM_0060_T06`, `GM_0081_T04` ; femme `GF_Kromede_01`,
  `GF_0005_T05`, `GF_0039_T05`. Le gabarit des boutons a un 5ᵉ emplacement. La table de l'écran de création
  (`LobbyCreateCharItem.dat`) est chiffrée : cette liste vient des icônes présentes et de l'observation du client.

## Supposé
- Le lien exact entre `RootTipRange` et le `RootTipAdd` du shader des cheveux n'est pas écrit dans les tables.
- Une septième zone de tatouage (le visage ?) expliquerait l'écart entre 6 zones dans `CMM_Body` et 7 dans l'onglet.
- `UT_Customizing_Costume_Current_Outfit` servirait à la re-personnalisation en jeu.

## Pour reproduire
1. Exporter `CustomizingUIListTable` et relever `ValueType`, `CompVariable` et `ColorVariable` de chaque bloc.
2. Lister les morph targets des têtes (`EarShapeChange`, `Eye_AO_*`).
3. Croiser `MaterialCustomizeParamSettings` avec les paramètres des instances `CustomMI`.
