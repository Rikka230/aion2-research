# Écran titre et salle de sélection

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : cartes `Preview_Login`, `Preview_Lobby_Temple_01`, `Preview_CreateChar` ; maîtres `EMM_AteriaCloud`,
  `EMM_AteriaRing`, `EMM_Simple_Fog` ; séquences `Cinematic/Login/Lobby/*` ; acts `Act/Common/lobby_character_*` ;
  acteur `AionEnvironmentVolume`
- **Méthode** : export JSON des cartes, composants, séquences et acts (CUE4Parse), shaders compilés (voir
  [shaders-compiles](shaders-compiles.md)), échantillonnage des pistes de séquence

## Établi

### Écran titre (`Preview_Login`)
- Les 60 objets de la scène sont fixes et aucun vertex shader ne lit le temps : ni la planète ni les astéroïdes ne
  tournent. Le mouvement vient :
  - des matériaux : défilement d'UV des anneaux, flowmap des nuages, lueur du sol qui défile ;
  - d'un tremblement permanent de la caméra, `Common_CameraShake_Loop_soft` (bruit de Perlin de 0,45° à 0,25 Hz) ;
  - de la séquence `Cinematic/Login/Lobby/Intro`, qui fait apparaître elle-même 8 objets (`COM_CliffBH01`, `COM_RockAB01`)
    animés par clés linéaires et cubiques, et porte la trajectoire, la focale et la profondeur de champ de la caméra.
- Nuages `Atreia_D_02_MI` (maître `EMM_AteriaCloud`) : f = `T_FlowTexture`.rg − 0,5 ; t = `FlowSpeed` (0,05) × temps ;
  deux lectures de `T_BaseColor` décalées de frac(t) × f et frac(t + 0,5) × f, mélangées par |2 frac(t) − 1| ; cycle de
  20 s.
- Anneaux (`EMM_AteriaRing`) : UV × `Scale` + temps × (`Speed`, 0), `Speed` = 0,025, 0,015 et 0,02.
- Les deux maîtres sont **éclairés** (DefaultLit, translucidité `TLM_Surface`), pas unlit. Les masques des anneaux sont
  dans le canal **B** (alpha blanc). Certains maillages ont toutes leurs couleurs de sommet à 0.
- La texture de flux est bleutée : un import automatique la prend pour une normale. Il faut la compression par défaut et
  sRGB coupé.
- Sol de la planète : le bruit émissif rouge n'existe que la nuit (intensité de jour nulle).
- Caméra du titre : la première de `Cam_Title/Cam_SelectRace`.

### Environnement des scènes
- `AionEnvironmentVolume` porte le soleil (direction, couleur, intensité), les cubemaps (ambiante `T_AO_Cube_02`), le
  post-traitement (LUT), `FogInscatteringColor` et une exposition automatique de 0,2 à 2.
- Les variantes de matériaux (sols, globes, effets) sont portées par les **OverrideMaterials** des composants de la carte :
  sans elles, le globe de la salle (`LOB_AtreiaA02`) est noir. Globe cristal : `ET_Crystal_Space`, `ET_Default_Crack_01`,
  `ET_Perlin_Noise_M`.
- Brume `EMM_Simple_Fog` : éclairée et translucide, éclairage volumétrique directionnel par sommet, distances et angle de
  fondu ; les instances du titre ont `BaseDay_Blend_FogColor` = 1 (tendent vers `FogInscatteringColor` du volume).
- Couleurs des projecteurs stockées en octets sRGB.
- La salle a quatre éléments tournants (`RotatingMovement`) ; le décor n'a **pas de collision**.

### Salle de sélection (`Preview_Lobby_Temple_01`)
- Chaque `BP_SelectCharPoint` porte `SlotPos`, `CameraTag SlotN`, `IntroActName lobby_character_enter_N`,
  `IdleMotionName LobbyChar_N`, `SelectActName lobby_character_select_N`, `EnterActName lobby_character_leave_N`
  (le champ « Enter » pointe sur « leave »). Neuf caméras portent les tags `Slot1…9` ; vue d'ensemble = caméra 7.
- Gestes `Character/Player/G?/Ani/Lobby/0N/` par emplacement : `Enter` (4 s, le bassin part de 2,74 m en arrière),
  `Idle` (12 s), `Select_Start` (2,7 s) puis `Select_loop` (15 s), `Leave` (4,5 s, marche de 3,4 à 5 m). Déplacement porté
  par `Bip001`. Variantes par arme : `G?_<SW|MA|BW|DA|ST|MB>_LobbyIdle`, présentation `G?_GS_Preview` (14 s).
- Séquences `Cinematic/Login/Lobby/Seq_short/Cam_Ch_0N_Leave_short` (2,33 s, fondu au noir de 1,73 à 2,27 s), visant
  `BP_SelectCharPoint1…8`.
- Départ : `Cinematic/Login/Lobby/Cam_Lobby_To_Tutorial` (2,3 s) : battants `LOB_DoorA01` / `LOB_DoorA2` de 90 à 190° et
  de −90 à −190° en lacet, caméra de 12 mm qui avance vers la porte, fondu au noir de 1,8 à 2,3 s ; sons
  `SEQ_Cam_Ch_Leave_Door` / `_Whoosh`.
- Pages d'interface : `Lobby_Login_BG`, `_Intro` (`Img_Logo`, carré de 512 px), `_Logined`, `_SelectRace`,
  `Lobby_SelectChar`, `Lobby_CreateChar`, `Popup_InputName`.

## Supposé
- Écart non expliqué : 8 points `BP_SelectCharPoint` et 8 séquences courtes, mais 9 caméras et 9 emplacements.
- Brume de la salle : deux lectures ont donné des valeurs différentes (densité 0,1 et extinction × 25 ; densité 0,15 et
  extinction 20), peut-être sur deux volumes différents. Dans les deux cas, ces valeurs copiées dans Unreal 5 récent noient
  la scène : le calcul de la brume d'AION 2 diffère.
- Deux grands astéroïdes posés restent invisibles à la reproduction : échelle de l'asset à vérifier.

## Pour reproduire
1. Exporter `Preview_Login` et vérifier qu'aucun composant de rotation n'anime la planète.
2. Désassembler `EMM_AteriaCloud` et retrouver `FlowSpeed` et le double échantillonnage.
3. Exporter un `BP_SelectCharPoint` et suivre ses acts jusqu'aux gestes `Lobby/0N/`.
