# Ciel, étoiles, aurores, baleines

- Étoiles : `Sky_Star01` (voir [textures-masques](textures-masques.md)). Les fichiers `T_Stars_*` et `T_Starfield_*`
  sortent en 443 × 221 (HDR) : trop petits pour un ciel net, peut-être des vignettes de cubemaps.
- Aurores : fonction `EMF_CurtainAurora` et masque `ET_CurtainAurora_M` ; panorama `T_skybox_aurora_02` (avec arbres en
  premier plan, inutilisable tel quel).
- Baleines du ciel : `Character/VEH/Animal/SkyWhale/SkyWhale_01` (monture, plumage clair) et `SkyWhale_02` (fantôme,
  texture `_DO` avec alpha) ; squelette de `SkyWhale_01`, vol plané `VEH_SkyWhale_01_Glide_Loop_Normal`.
- Raie du ciel : `Character/Monster/Animal/Skyray/Skyray_01`, vol `MOB_Skyray_01_Run`. Pose de liaison « debout » :
  lire l'orientation sur la pose animée.
