# Cheveux : structure des coiffures et matériau CMM_Hair

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `Character/Player/GF/Hair/GF_Hair_*`, `Character/Player/GM/Hair/GM_Hair_*`, maîtres `CMM_Hair`,
  `CM_HairCard`, `CM_HairCardCap`, atlas `T_Hair_0x_DOI`, catalogue `HairType`, vignettes `UT_Customizing_Thumbnail_Hair_*`
- **Méthode** : export glTF des coiffures (couleurs de sommet, os), paramètres des instances et shader compilé de
  `CMM_Hair` (voir [shaders-compiles](shaders-compiles.md)) ; comparaison de rendus aux vignettes officielles

## Établi

### Géométrie
- Une coiffure est un maillage skinné de 4 000 à 30 000 triangles, fait de **cartes de mèches** (couleur de sommet G = 1,
  atlas `T_Hair_0x_DOI`) et parfois d'une **calotte** (G = 0, texture `ROI_HairCap`). Couleur de sommet R : 0 à la
  racine, 1 à la pointe.
- Chaîne d'os : `Root … Bip001-Head > Hair_root > Hair_*`. `Bip001-Head` est présent dans les squelettes de tête et de
  cheveux mais pas dans celui du corps. Certaines coiffures ont des os qui ne suivent pas le préfixe `Hair_`
  (`GM_Hair_105` : `LB_Hair_*` sous `Hair_top`) : un import sur un squelette partagé refuse alors la fusion
  (« cannot merge bone tree »). Les coiffures courtes de PNJ n'ont pas d'os de mèche.
- Les variantes `_Type1` / `_Type2` d'une coiffure ont la même taille : ce sont des ajustements pour le casque.
- Les barbes (`GM_Beard`, 12 maillages, `MI_GF_Beard`) appartiennent à la famille des cheveux et se portent en
  Leader Pose sur la tête.

### Atlas et instances
- `T_Hair_0x_DOI` : R = couleur, G = opacité. **Un atlas par coupe**, à lire dans l'instance (`Card_Index_Texture`) :
  par exemple 114, 010 et 416 utilisent `T_Hair_05`, 312 utilise `T_Hair_01`.
- 96 instances de cheveux, dont 9 à calotte (ex. `GF_Hair_408`, `GF_Hair_001` : 826 sommets de calotte,
  `T_Hair_01_DOI_Cap01`, paramètre `CapRootUseVertexColor`).
- Les instances n'ont pas la même teinte par défaut (calotte de `GF_Hair_001` rouge sombre, cartes de 312 presque noires).

### Calcul de `CMM_Hair` (shader compilé)
- La direction du fil passe par la broche **Normale** : c'est la seule que lit le modèle d'ombrage Hair.
- Opacité tramée (masque d'opacité en dither), seuil 0,3333, adoucie selon la distance (`OpacityFar` 1,8, biais de mip −1).
- Couleur × identifiant × 0,8 ; AO 0,6 ; spéculaire 0,6 × identifiant ; rugosité selon la distance et l'identifiant ;
  Backlit = 1 (CustomData0) ; Scatter 0,9.
- Deux tons : dégradé = saturate((racine − 0,5 + RootTipAdd) × RootTipContrast + 0,5) ; couleur bornée à [0,003 ; 1].
- Calotte : `FlowMap_HairCap`, lue en UV2 et ajoutée à la tangente, seulement quand round(VC.G) = 0.

### Catalogue et vignettes
- 185 coiffures au catalogue visuel. Les vignettes officielles (95 relevées, masculines) font 256 × 256 sur fond
  transparent, mannequin gris sans visage (`GM_Hair_000`, la coiffure « sans cheveux »), vue de trois quarts :
  lacet 10°, tangage 12°, zoom 1,16. Ce cadrage recouvre le masque des cheveux à 0,74 (IoU), contre 0,49 de face.
- Aucune queue de cheval de côté dans le catalogue.

## Supposé
- Les instances masculines partageraient l'atlas de l'instance féminine de même numéro ; peu d'instances masculines
  ont été vérifiées.
- Les vignettes féminines existent peut-être ailleurs dans les archives ; non trouvées.
- Un écart de saturation sur les couleurs sombres très saturées subsiste en reproduction ; il pourrait venir du modèle
  Hair d'Unreal 5 plutôt que du calcul.

## Pour reproduire
1. Exporter une coiffure en glTF et colorer ses sommets par G (cartes / calotte) et R (racine / pointe).
2. Lire `Card_Index_Texture` dans l'instance pour trouver l'atlas.
3. Dans Unreal 5, brancher la tangente du fil sur la broche Normale d'un matériau Hair, opacité en dither à 0,3333.
