# Interface : HUD et fenêtres principales

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `UI/Widget/HUD/*` (`HUD_Layer`, `HUD_Menu`, `HUD_QuickSlot*`…), fenêtres `FWindow_*` et `Popup_*`
- **Méthode** : export JSON des widgets et de leurs dépendances (CUE4Parse), reconstruction dans Unreal 5 ; comportement
  observé sur des vidéos publiques (marqué « observé »)

## Établi

### HUD
- `UI/Widget/HUD` : 45 widgets ; `HUD_Layer` compte 8 535 éléments (409 matériaux, 44 émetteurs, 308 `ScaleBox`). Sa
  construction complète demande ≈ 940 paquets de dépendances, 5 atlas et 422 textures.
- Blocs nommés `<ordre>_EditableBox_<rôle>` : `100_EditableBox_Chat`, `105_EditableBox_Gauge`, `120_EditableBox_Menu`,
  `121_EditableBox_MenuJournal`, `123_EditableBox_MenuInventory`, `130_EditableBox_Currency`, `137_EditableBox_Fly`,
  `200_Size_QuickSlotListGroup`, `400_EditableBox_GetItemList`. `UI_HudEditDataTable` ne donne aucune position : c'est le
  gabarit qui fait foi.
- Raccourcis : `HUD_QuickSlotHolder` + `HUD_QuickSlotSkill`, cases de 70 × 70 ; touche, recharge, chaîne et voile natifs ;
  chaque raccourci contient un second bouton caché (`Competence/P_StateIcon/P_Btn_Skill`). Rangée du bas à 10 px du bord.
- XP : `190_UserInfo/P_Img_EXPGauge`, matériau `MI_UI_GaugeFlying_RounedRect_2` ; la jauge suit le contour de l'emblème de
  classe. Jauge de vol : `Flying/P_Img_FpGauge` (circulaire, dans le bouton des ailes).
- Incantation : cible `P_TargetInfo/P_SkillCasting` (racine `Size` repliée par défaut) ; joueur `HUD_Layer/SkillCasting`
  dans `105_EditableBox_Gauge` ; aussi `ItemCasting`, `InteractionGauge`, `GatherProgress` (20 animations).
- Discussion : 15 lignes `Common_ChatListItem*/P_RTxt_Msg`, saisie `P_Chat/P_ETxt_Input`.
- Mini-carte : cadre `MiniMap/Over_Map`, flèche `P_PlayerIcon`. Carte du monde `FWindow_WorldMap` : 428 sprites, 22
  animations, fond = rendu 3D en direct (`P_RenderTargetBackground`) ; bouton `P_Btn_Filter`.
- Plaques de nom `NamePlate`, `NamePlateSimple`, butin `NamePlate/HUD_DropItemNamePlate` ; étoiles de quête
  `UT_NP_QuestHero_Acquired` / `_Complete`.
- Chiffres de dégâts : `Floater_Normal_Sprite`, style `DamageFloater_Default` de `UI_TextStyleDataTable` : police grasse
  de 28, contour noir de 1, texte penché de 5° ; MultiHit en 24, Buff `7CCDE1`, DeBuff `FD5153`.
- Autres : `Popup_Death`, `HUD_GetItem` / `HUD_GetItemListItem`, `KeyGuide`, `Ping`, `Common_LevelUp`, `Common_QuestMsg`.

### Menu général `HUD_Menu`
- 47 entrées à icône et 6 accès rapides (53). Icônes dans la propriété `Texture` de chaque instance de l'export brut.
  Certaines entrées sont cachées ou repliées dans les données (`P_Collection`, `P_Emblem`, `P_WorldTrade`, `P_Housing`,
  `P_Pantheon`). L'entrée `P_Customizing` ouvre le barbier (`FWindow_Customizing`).

### Fenêtres
- Inventaire (cube) : `Popup_Inventory` = `VBox_Inven` [`P_InventoryList` (`Common_Inventory`) + `HBox_Bottom`] et
  `P_EquipInfo` (`Popup_EquipInfo`). Colonne d'équipement 220 × 1125 (26 cases sur 13 rangées : main, seconde main,
  casque, torse, épaules, gants, jambes, bottes, boucles, collier, anneaux, ceinture, cape, et accessoires) ; liste de
  470 px de large ; fond `UT_Common_SideBG` ancré à droite. Rareté par `MI_UI_ItemGrade`. Cases `Slot_InventoryItem`.
- Compétences : `FWindow_Skill`, `FWindow_SkillListItem` (ListView) ; `P_SizeBoxViewPoint` montre le personnage qui
  exécute la compétence ; schéma de portée `Img_Effect_Range_ArcTriangle`.
- Journal des quêtes : `FWindow_Journal` (`_TabList`, `_Quest`, `_QuestObjectiveListItem`), catégories `P_Hero`,
  `P_District`, `P_Duty`, `P_Ascension`.
- Dialogue : `FWindow_NPCTalk` (`_QuestInfo`, `_QuestListItem`, `_FunctionListItem`), caméra `NPCTalk_FocusCameraTalk`
  (plan buste, devant le PNJ, côté joueur).
- Marchand : `FWindow_NPCShop` (5 pages, 35 widgets) ; boutique `FWindow_BMShop_Preview` ; teinture
  `FWindow_Skin_DyeList`, `Slot_DyePalette`.
- Social : `Popup_Community`, `FWindow_PartyDungeon`, `FWindow_Post`, menu contextuel
  `Popup_CommunityPlayerFuncTooltip` (5 actions). Marché : `FWindow_Trade` (vente, règlement, historique).
- Butin : `Popup_DropsReward` / `_ListItem`. Confirmation : `Popup_Message_Patch`.
- Gabarit de bouton commun : `NamedSlot_HotKey`, case d'icône `P_Size_Icon` de 44 × 44, `P_Txt_SubText` ; styles
  `Disabled`, `Dimmed`. Le gabarit Popup a un voile plein écran.

### Écran de chargement
- `UI/Widget/Common/Common_LoadingScreen` : fond `UT_BG_Loading` (2048 × 2048), cadre 2380 × 1170 en `ScaleToFill`,
  indicateur de cinq points de 6 × 6 en bas à droite, marges de 100. Variantes `_Glyph` (sept textures de lettres, classe
  native `AionImageThrobber`) et `_ForMovie`. Musique : voir [sons-wwise](sons-wwise.md).

## Observé (vidéos publiques)
- Cube ancré à droite sur toute la hauteur, grille de 7 colonnes, compteur de capacité ; menu général en panneau opaque
  à droite sur trois colonnes ; HUD : vie et mana autour de l'emblème en bas au centre, deux rangées de raccourcis.

## Supposé
- La taille de 60 px des cases du cube est attestée dans une ligne de butin, pas mesurée dans le cube lui-même.
- `Popup_DropsReward` serait la fenêtre de butin d'un cadavre ; non prouvé.

## Pour reproduire
1. Exporter `HUD_Layer` avec ses dépendances et relever les blocs `*_EditableBox_*`.
2. Exporter `Popup_Inventory` et mesurer `P_Size` et `VBox_Inven`.
