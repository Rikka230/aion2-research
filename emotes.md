# Émotes (SocialAction)

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `Act/Player/02_Ani_Common/11_SocialAction`, `Character/Player/G?/Ani/SocialAction/`,
  `Character/NPC/SocialAction/`, `FX/Particle/PC/Common/SocialAction/`, `WwiseAudio/Events/PC/*/Social`,
  `UI/Widget/Popup/Popup_*SocialAction*`
- **Méthode** : index des chemins contenant « Social » (≈ 5 000 fichiers), export JSON des acts et montages, décodage des
  médias Wwise

## Établi

### Arborescence
- Acts joueur : `Act/Player/02_Ani_Common/11_SocialAction` : 96 `_Design`, 96 `_Fx`, 96 `_Sound` ; sous-dossiers
  `Weapon/` et `CLB01/` (collaboration).
- Animations joueur : `Character/Player/G?/Ani/SocialAction/` : séquences `_Start`, `_Loop`, `_End` ; `AM/` (montages,
  ex. `GF_Fly_Social_Cool_AM`) ; `Fly/` ; `Portrait/` (`GF_PortraitPose_003_*`) ; `BM/` ; `CLB01/` ; `Dance/` ;
  `Weapon/<famille>` ; `WingA` à `WingF`.
- PNJ : `Character/NPC/SocialAction/{Adult,Kid,Elder}/{GF,GM}`, `Shugo`, `Shulack`, `Lycan/{LF,LM}` ; acts PNJ dans
  `Act/NPC/Humanform/Common/NPCSocialAction`. Monstres : `Act/Monster/<Nom>/<Nom>_Die_SocialAction`.
- Volume : 268 séquences GF et 294 GM (animations d'ailes comprises), 117 entrées au catalogue côté joueur.
- Chaque émote a trois phases (sections `Start`, `Loop`, `End`). Exemple `Fly_Social_Cool` : GF 1,333 / 1,467 / 1,0 s,
  GM 3,0 / 2,933 / 2,667 s. Une section `End` peut ne renvoyer à aucune séquence propre (segment quasi nul en fin de
  boucle, ex. `Fly_Social_Weapon_BW`).
- Tâches les plus fréquentes dans les acts d'émotes : `FootstepFX` (1 026), `Particle` (1 008),
  `MaterialDynamicParameter` (128), `Snapshot` (70), `PostProcMaterial` (64), `AkPostEvent` (32). Voir
  [acts-et-effets](acts-et-effets.md).
- Effets : 17 systèmes référencés (`P_Flare_*`, `P_Social_Weapon_Cast_000` à `011`, `P_Aura_238_*`, `P_RadialBlur_102`),
  modules dans `/Game/FX/Particle/Module/{Flare,RadialBlur,Light,LensFlare}`.

### Sons
- Voix : `WwiseAudio/Events/PC/PC_Vo_1P/Social/{GF,GM}` (48 chacun) et `PC_Vo_3P/Social/{GF,GM}` (44 chacun), schéma
  `G?_Social_<Nom>[_NN][_Start|_Loop|_End]_{1P,3P}`. Mouvements : `PC_1P/Motion_1P/Social`, `PC_3P/Motion_3P/Social`
  (22 chacun). Son d'arme : `PC_Social_Weapon_1P` / `_3P`.
- Plus de 1 500 médias liés aux émotes se décodent avec vgmstream.

### Interface
- `Popup_ChatSocialAction`, `Popup_Chat_SocialActionTooltip(_ListItem)`, `Popup_SocialActionSetting(List|_ListItem)`,
  `Popup_SocialAction_QuickMenu(_ListItem|_Tooltip)` ; mobile `UI/Widget_M/HUD_M/HUD_ActionButton_SocialAction_M` ;
  carte d'aperçu `UI/Map/Preview_SocialAction.umap`.
- Sprites d'onglets `UT_Common_SocialAction_Tab_{Action,Communication,Emotion}_Sprite`.
- Dans l'arbre, le voile d'infobulle est dans `P_TooltipPanel`, qui contient aussi le panneau principal : masquer le
  voile seul.
- L'accessoire `SA_CLB_01`, cité par les émotes de collaboration, est absent des archives.

## Supposé
- 1P serait le son entendu par le joueur qui joue l'émote, 3P celui des observateurs.
- Les adaptations nécessaires d'un genre à l'autre suggèrent que certaines émotes n'existent que pour un genre.
- Les icônes `Icon_BMShop_Social_*` indiquent qu'une partie des émotes vient de la boutique.

## Pour reproduire
1. Lister `11_SocialAction` et vérifier la triple présence `_Design` / `_Fx` / `_Sound`.
2. Exporter un montage `AM/` et lire ses sections et sa durée.
3. Suivre les `AkPostEvent` jusqu'aux évènements `_1P` / `_3P`.
