# Acts d'AION 2 : actions, effets, sons, cinématiques

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `/Game/Act/` et ses miroirs `FX/Act/` et `Sound/Act/` ; types `AionAct`, `AionActContainer`, tâches
  `AionActTask_*` ; `Cinematic/Cut_Scene/`
- **Méthode** : export JSON des acts et des séquences (CUE4Parse), liste des chemins

## Établi

### Organisation
- Une action (compétence, esquive, émote, interaction, mort de monstre, cinématique) est un **act** : un `AionAct` ou
  `AionActContainer` principal et son `<Nom>_Design` dans `Act/…`, plus un `<Nom>_Fx` dans l'arborescence miroir
  `FX/Act/…` et un `<Nom>_Sound` dans `Sound/Act/…`.
- Dossiers vus : `Act/Player/04_Normal/`, `Act/Player/05_Gladiator/GS/`, `Act/Player/02_Ani_Common/03_GS/{01_Fly,03_Base}/`,
  `Act/Player/02_Ani_Common/11_SocialAction/`, `Act/Common/` (lobby), `Act/Monster/<Nom>/`, `Act/NPC/…`,
  `Act/Cinematic/Cut_Scene/<zone>/Seq_*`.
- Les tâches portent `ActivationTime`, `TaskLinks`, `OwningSectionName` (Start / Loop / End) et des courbes internes
  `AionActContainerCurve`, adressées comme `<act>_Fx.<index>`.

### Tâches rencontrées
- `AionActTask_Animation` (montage, branches par genre et par état de déplacement : voir
  [animations-combat](animations-combat.md)).
- `AionActTask_EndureStart` / `AionActTask_EndureEnd`.
- `AionActTask_Particle` : `FXSystemAsset`, `TimeScale`, `Duration`, décalages de position, rotation et caméra,
  `DepthFade`, `ColorMult`, `MeshType` (`ECharacterSubMesh::MainWeapon_MainMesh`, `MainWeapon_SubMesh`),
  `PointInheritType` (`PosRotOnly`, `PosOnly`), `AttachPoint`, `KillType` (`ForceFade`, `Immediately`),
  `ForceFadeOutDuration`, `VisiblePriority`, `VisibleDistance`, `FXVisibleRelationType`, `bLocalPlayerOnly`.
- `AionActTask_Snapshot` : fait apparaître un accessoire squelettique (`SkeletalMeshAssetPtr`, `MaterialInstance`) sur
  un point d'attache, avec un paramètre `forcefade` piloté par courbe. C'est ainsi que les gestes posent leurs objets.
- `AionActTask_PostProcMaterial` : matériau de post-traitement temporaire (ex. `MI_PP_LensFlare_004`, fondu d'entrée 1,5,
  de sortie 0,5, durée 0,35, joueur local seulement).
- `AionActTask_MaterialDynamicParameter` : préréglages `AbnormalRim`, `Custom` ; paramètres `FX_AbnormalRim_*`,
  `FX_AbnormalFlicker_*`, `forcefade`.
- `AionActTask_AkPostEvent` : évènement Wwise, avec `bUse3PEvent`, `Event` et `Event_3P` (voir [sons-wwise](sons-wwise.md)).
- Notifies dans les montages : `AionAnimNotify_AkPostEvent`, `AionAnimNotify_FootstepFX` (`FootBoneName` `RFootStep` /
  `LFootStep`), `AionSetMovingAttackSkippable`.

### Exemples
- Interaction de gravure (`Act/Player/04_Normal/ArtifactInteraction*`) : effets `P_Flare_020/021`, `P_Glow_016/022`,
  `P_SmoothFlicker_Light` ; gestes `GF_Engraved_Start/_Loop/_End` ; statue `Kisk_L_01` et `Kisk_L_01_Interaction`.
- Butin : `AddDropEffect_<qualité>_Fx` (`P_AddDropEffect_*`, `P_Glow_032`, `P_Flare_021`, `P_Light`).
- Montée de niveau : act `Level_Up` = `Level_Up_Fx` + `Level_Up_Sound` (vide). `Level_Up_Fx` compte 18 tâches de
  particules (`P_Aura_015_20` avec les arcs `SM_Arc_009`, flares sur `SM_Plane_005`, fragments, étincelles, flou radial,
  lumière). Le son est joué par la fenêtre `Common_LevelUp` (`UI_Signal_LevelUp`) ; un Niagara d'interface existe
  (`NS_UI_Levelup_2`).
- Bibliothèque d'effets : textures et maillages rangés par élément (≈ 940 textures et 170 maillages relevés).

### Cinématiques
- **154** LevelSequence « Main » (149 scènes) dans `Cinematic/Cut_Scene/<zone>/Seq_<n>/` (sous-dossiers `Ani`, `FX`,
  `Light`). Sequencer standard : CineCameraActor, piste de découpage caméra, secousses, acteurs apparaissants,
  attaches, morph targets, Control Rig du visage `CR_GF_FaceRig`, pistes de matériaux.
- Elles sont pilotées par des `AionActContainer` (séquence, `_Design`, `_Fx`, `_Sound`).
- Les vidéos passent par une LevelSequence à piste média (ex. `QM1001_Movie` : `FileMediaSource` vers un `.mp4`, piste
  de sous-titres `AionMovieSceneSubTitleTrack`, son Wwise, fondu). 61 vidéos sont indexées ; les `.mp4` sont hors des
  archives.

## Supposé
- Le sens d'« Endure » (super-armure ?) et l'unité exacte d'`ActivationTime` (secondes ou fraction) ne sont pas établis.
- **Type des systèmes `P_*`** : deux lectures divergent. Une analyse des effets d'émotes classe 18 systèmes `P_*` comme
  Niagara cuits ; une autre, sur les effets d'interaction et de butin, les traite comme Cascade. Le préfixe ne dit pas
  le type : lire la classe de l'export (`NiagaraSystem` ou `ParticleSystem`) avant de conclure. Dans les deux cas, le
  graphe source n'est pas disponible.

## Pour reproduire
1. Exporter un act et son `_Design` en JSON ; lister les tâches et leurs `ActivationTime`.
2. Suivre `<act>_Fx` dans `FX/Act/` et `<act>_Sound` dans `Sound/Act/`.
3. Vérifier la classe de chaque `FXSystemAsset` référencé.
