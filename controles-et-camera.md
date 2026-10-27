# Contrôles et caméra

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : énumérations et structures du `.usmap` (`EAionControlMode`, `EAionInputAction`…), structure
  `DefaultSettingsTableItem`, DataAsset `/Game/UnrealData/CameraDataAsset` (`PlayerCameraDataAsset`),
  `AionCharMovementComponent`, fenêtre `FWindow_Setting`
- **Méthode** : lecture du `.usmap` (noms et types), export JSON du DataAsset de caméra et de la fenêtre Options
  (CUE4Parse) ; comportement croisé avec des guides publics datés (marqué « observé »)

## Établi

### Modes de contrôle
- `EAionControlMode` {Action, Classic} (et `EClientControlMode`). Les réglages existent en `_Mode1` (Action) et `_Mode2`
  (Classique), plus `_Mobile` et `_Gamepad`. Le mode est transmis au serveur avec les options du client.
- Champs de `DefaultSettingsTableItem` : visée (`AimHeight_Mode1`, `AimFilterType_Mode1`, `bShowAimTargetDistance_Mode1`),
  ciblage (`AutoSkillTargetingType_*` {On, IgnoreLongPress, Off}, `TargetSearchCriteria` {Character, Camera, None},
  `bSetMainTargetByAttack/LeftClick/RightClick_Mode1`), souris du mode Classique (`MoveToLocationInputType_Mode2`,
  `ObjectClickedActionInputType_Mode2`, `bMouseLeftClickOnlyCameraRotate_Mode2`, `bLockMouseCursor_Mode2`), caméra
  (`PlayerCameraMode`, `CameraDistance`, `CameraSpeedYaw`, `CameraSpeedPitch`, `bEnableBattleCameraShake`,
  `bEnableCameraTargetLock`), divers (`KeepAttackAtMove`, `bShowWeaponInPeaceState`, `SkillReservation`,
  `AutoPowerSavingModeSeconds`). Les valeurs par défaut sont dans une table chiffrée.

### Entrées
- Il n'y a **pas d'assets Input Action** à extraire : `AionInputSystem` les crée à l'exécution depuis des tables chiffrées
  (`InputAction`, `KeyGuide`, `SkillControlSetting`).
- `EAionInputAction` (≈ 200 entrées) : `Move_Forward`, `Move_Right`, `Move_Jump`, `Move_Fly`, `Move_FlyUp`, `Move_FlyDown`,
  `Move_Ride`, `Move_Sprint`, `Move_Walk`, `Move_AutoForward`, `Glide_Boost`, `Glide_DashUp`, `Glide_Rotate`,
  `Camera_RotateX/Y`, `Camera_Zoom`, `Camera_Mode`, `Camera_ToggleLockOn`, une entrée `UI_*` par fenêtre, `Combat_*`,
  `Casting_Cancel`, `Target_Cancel`.
- Contextes `EInputEventContext` {Move, Glide, UIMode, FullScreen, Modal, Tutorial, Lock, Racing, UIGuide,
  PowerSavingMode, Shooting} ; déclencheurs {Press, PressRelease, Axis} ; états du guide des touches `EKeyGuideState`
  {Run, Fly, Glide, Swim, Dive, Fall, WindPath, Racing, Dive_NoWing, Fall_NoWing, JumpJump, Masquerade}.
- Emplacements numérotés : 8 compétences, 8 objets, 4 stigmas, 2 compétences d'évènement, 4 emplacements de compétences
  enchaînées, 4 de métamorphose ; 5 préréglages d'équipement, 3 de compétences ; 18 marqueurs de cible.

### Caméra (`PlayerCameraDataAsset`)
| Mode | FOV | Tangage | Distance défaut / min / max (cm) | Décalage (cm) | Tangage limite |
|---|---:|---:|---|---|---|
| FreeView | 90 | −8 | 300 / 150 / 1800 | (0, 0, 100) | ±80 |
| ShoulderView | 90 | −8 | 300 / 150 / 1800 | (0, 80, 100) | ±80 |
| TopView | 90 | −80 | 1500 / 1800 / 1800 | 0 | −80 |
| SideView | 70 | −8 | 500 / 200 / 1800 | (0, 0, 120) | 0 |

- Sensibilité souris 0,4 (lacet) et 0,35 (tangage) ; manette 3,5 et 2,2.
- `RefBoneNameForCustomizingDiff` = `Bip001-Head` (la caméra suit la morphologie) ; lissage de distance ;
  `ZoomChangeAdjustSpeed` 1000 ; verrouillage de cible : distance minimale 300, retard de rotation 5 ;
  `RotationLagSpeed` 100 ; sous l'eau, tangage −10° en 0,7 s ; fondus de sous-zone et d'évènement 3 s.
- Surcharges par état : planage et vol FOV +10 avec retard de position ; courant aérien (Windpath) FOV +40 ;
  accélération de planage : retard vertical seul, 80 cm (100 en piqué).
- Mouvement : `AionCharMovementComponent` (dérivé de `CharacterMovementComponent`), `EOrientRotationType` {Stop,
  OrientRotationToAcceleration, ToVelocity, ToDesired, ToCamera}, `EMoveMode` {Walk, Run, Sprint}.

### Fenêtre Options
- `FWindow_Setting` (327 widgets) : onglets accessibilité, compte, audio, combat, données, graphismes, infos, annonces,
  touches. Pages `_Graphic`, `_Audio`, `_SliderOption`, `_ComboBoxOption`, `_CheckBox`. Panneaux `P_Pan_Mode1` /
  `P_Pan_Mode2` pour les deux modes de contrôle.

## Observé (guides et vidéos publics, non lu dans les données)
- Mode Action par défaut : curseur caché (Alt l'affiche), la souris tourne la caméra sans clic, déplacement relatif à la
  caméra, le personnage se tourne vers son déplacement. Mode Classique : curseur visible, clic au sol, deux boutons
  pour avancer. Le changement de mode se fait aussi en jeu.

## Supposé
- Les champs déclarés mais absents de l'export de caméra (ex. `TargetLockTargetDistanceMax`) seraient à leur valeur par
  défaut de classe.
- Le bouton qui tourne la caméra en mode Classique : les guides se contredisent.

## Pour reproduire
1. Chercher `EAionControlMode`, `EAionInputAction` et `DefaultSettingsTableItem` dans le `.usmap`.
2. Exporter `/Game/UnrealData/CameraDataAsset` en JSON et lire chaque mode.
