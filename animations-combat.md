# Animations de combat : montages, esquives, banques par arme

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : acts `Act/Player/05_Gladiator/GS/Gladiator_Skill00*`, `Act/Player/04_Normal/NormalDodge_*`,
  `Act/Player/02_Ani_Common/03_GS/*/GS_CombatDodge_*` ; montages `Character/Player/G?/Ani/<Classe>/<Arme>/AM/`
- **Méthode** : export JSON des acts, `_Design` et montages (durées, sections, notifies, slots) ; index des chemins

## Établi

### Branches d'un act de compétence
- Champs : `AnimMontageAssetPtr` (pointeur générique), `BranchByGender`, `BranchByMoveState`,
  `AnimMontageAssetForMale`, `AnimMontageAssetForFemale`, `AnimMontageAssetForMale_Move`,
  `AnimMontageAssetForFemale_Move`.
- Les branches `_Move` pointent vers des montages suffixés `_U_AM` (haut du corps, ex.
  `GF_Gladiator_GS_Skill001_U_AM`), joués pendant le déplacement.
- Le pointeur générique peut contredire les branches : `Gladiator_Fly_Skill001` référence des montages de vol par genre,
  mais son pointeur générique reste le montage terrestre. Il faut choisir la branche, pas le premier chemin trouvé.

### Trois horloges dans un montage (Gladiateur, espadon, GF)
| Geste | Durée du montage | Notify `AionSetMovingAttackSkippable` | `EndureEnd` (Design) |
|---|---:|---:|---:|
| Skill001 | 1,933 s | 1,1 s | 0,59 |
| Skill002 | 2,117 s | 1,4 s | 0,474 |
| Skill003 | 1,633 s | 0,9 s | 0,338 |

- Les trois montages terrestres sont sur le slot `UpperBody` ; `EndureStart` démarre à 0. Le montage en vol
  (`GF_GS_Fly_Skill001_AM`) est sur `DefaultSlot`.
- Aucune de ces valeurs ne donne l'instant d'impact ni une durée d'invulnérabilité. Ne pas confondre `EndureEnd`, le notify
  d'annulation en mouvement et l'impact.
- Sons : notifies `AionAnimNotify_AkPostEvent` dans les montages.

### Esquives à huit directions
| Act | Montages GF |
|---|---|
| `04_Normal/NormalDodge_Ground` | `GF/Ani/Common/NoWeapon/AM/GF_NoWeapon_Dodge_<dir>_AM` |
| `04_Normal/NormalDodge_Flight` | `GF/Ani/Common/NoWeapon/Fly/AM/GF_Fly_Dodge_<dir>_AM` |
| `03_GS/03_Base/GS_CombatDodge_Ground` | `GF/Ani/Common/GS/AM/GF_GS_Dodge_<dir>_AM` |
| `03_GS/01_Fly/GS_CombatDodge_Fly` | `GF/Ani/Common/ST/AM/GF_ST_Fly_Dodge_<dir>_AM` (GM : `GM/Ani/Normal/Fly/AM/GM_Fly_Dodge_<dir>_AM`) |

- Sections `Default`, `Default_FL`, `Default_FR`, `Default_L`, `Default_R`, `Default_B`, `Default_BL`, `Default_BR`.
- L'asymétrie GF / GM de l'esquive en vol est réelle dans l'index ; ne pas la « corriger » par substitution de noms.
- Aucune distance ni durée d'invulnérabilité dans ces exports.

### Nommage et banques
- Schéma observé : `<Genre>_<Classe>_<Arme>_Skill<NNN>[_U|_Fire|_Upper|_Start|_A3]_AM`. Écarts : montages
  `Skill0040_AM` / `Skill0041_AM` pour des séquences `Skill040` / `Skill041` ; séquences `Gladiator_Skill30_Smash` /
  `_Upper` sans préfixe de genre.
- 39 ensembles d'attente et de course par arme (épée, masse, dague, grimoire, orbe, bâton, arc, espadon, sans arme) ;
  courses diagonales natives (`GF_GS_Run_FL`, `_FR`, `_BL`, `_BR`) ; mort `GF_Die`, `GF_Die_Loop`.
- Gestes communs (retour, bandage) dans le dossier `Normal` ; une partie des incantations du grimoire dans un dossier `Old`.
- Squelettes `GF_RootMesh_Skeleton` et `GM_RootMesh_Skeleton`. Os d'effets vus : `Fx_Root_R_01`, `Fx_Root_L_02`, `fx_shot02`.

### Reproduction dans Unreal 5
- Un montage lié au squelette GF joué sur un corps GM ne déclenche pas d'erreur visible dans Unreal 5 récent : il faut des
  variantes par genre ou un reciblage.

## Supposé
- Les codes `BW`, `ST` et `GT` (vus dans les émotes et les esquives) désigneraient l'arc, le bâton et une troisième arme non
  identifiée.
- L'instant d'impact se déduit seulement du mouvement de l'arme (pic de vitesse) ; aucune donnée lisible ne le donne.

## Pour reproduire
1. Exporter `Gladiator_Skill001` et `Gladiator_Skill001_Design` ; relever les branches et `EndureEnd`.
2. Exporter les montages pointés et comparer durées, slots et notifies.
3. Lister `Dodge_` dans l'index pour retrouver les huit directions par famille.
