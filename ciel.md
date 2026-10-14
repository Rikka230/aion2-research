# Ciel, étoiles, aurores, baleines

## Étoiles : comment AION 2 les dessine (établi)

- Build : Steam 25767555. Méthode : lecture des données de matériau compilées (`CachedExpressionData`) de
  `BG/Common/Sky/Sky_Mat` avec CUE4Parse.
- `Sky_Mat` est construit sur **UltraDynamicSky** (dossier `UltraDynamicSky/` présent dans le contenu). Les étoiles
  passent par la fonction `UltraDynamicSky/Materials/Material_Functions/Stars`.
- Paramètres : interrupteur `Real Stars` = faux (la carte réelle `Real_Stars` 4096² n'est pas utilisée),
  `Stars_Texture` = `UltraDynamicSky/Textures/Sky/Tiling_Stars` (2048², motif d'étoiles à répéter),
  `Stars Color` = (0,596 ; 0,706 ; 1,0), `Stars Orientation` = (1, 0, 0), `Stars Orbit` = 0.
- `Sky_Star01` (voir [textures-masques](textures-masques.md)) n'est **pas** la source des étoiles du ciel principal.
- Les fichiers `T_Stars_*` et `T_Starfield_*` sortent en 443 × 221 (HDR) : sans doute des vignettes, inutilisables.

## Étoiles : reproduction qui marche (supposé fidèle, vérifié visuellement)

Lire `Tiling_Stars` en **projection triplanaire** sur la direction du regard (poids `pow(abs(d), 6)`), deux couches
à des échelles sans rapport simple (environ 3,1 et 5,3) pour casser la répétition, teinte `Stars Color`, léger
scintillement par une troisième lecture qui dérive lentement. À éviter : une seule image étirée en latitude-longitude
(étoiles en gros carrés) et la projection sur six faces de cube (arêtes visibles).
- Aurores : fonction `EMF_CurtainAurora` et masque `ET_CurtainAurora_M` ; panorama `T_skybox_aurora_02` (avec arbres en
  premier plan, inutilisable tel quel).
- Baleines du ciel : `Character/VEH/Animal/SkyWhale/SkyWhale_01` (monture, plumage clair) et `SkyWhale_02` (fantôme,
  texture `_DO` avec alpha) ; squelette de `SkyWhale_01`, vol plané `VEH_SkyWhale_01_Glide_Loop_Normal`.
- Raie du ciel : `Character/Monster/Animal/Skyray/Skyray_01`, vol `MOB_Skyray_01_Run`. Pose de liaison « debout » :
  lire l'orientation sur la pose animée.
