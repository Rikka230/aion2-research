# Interface UMG : organisation, échelle, pièges de lecture

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `/Game/UI/Widget/{FWindow,Popup,HUD,Common,Button,Slot,Lobby,NamePlate}`, `/Game/UI/Widget_M/` (mobile),
  `/Game/UI/Resource/{Texture,Sprite}`, `/Game/UI/Map/` ; réglage d'échelle `UIScaleCurve`
- **Méthode** : export JSON des Widget Blueprints et de leurs dépendances (résolution récursive, CUE4Parse), relevés sur
  58 à 82 pages, reconstruction dans Unreal 5 et vérification par clics réels (hit-test Slate)

## Établi

### Résolution et échelle
- L'interface est dessinée pour **2560 × 1440**. Poser ses blocs dans des cadres de 1920 × 1080 les décale.
- `UIScaleRule` = Custom ; `UIScaleCurve` : 720 → 0,667 ; 1080 → 1,0 ; constante au-delà (0,833 à 900 px). Une lecture
  « 1440 → 1 » est fausse.

### Composition des pages
- Sur 58 pages exportées : 584 matériaux d'interface (familles Trans, Blink, Flare, ItemGrade, Sprite_Mask, Line_Move,
  Retainer…), 147 émetteurs `UIParticleEmitterNiagara`, 391 `ScaleBox`, 110 `EditableBox`, 6 `RetainerBox`, 2
  `UIBackgroundBlur` ; contour de police sur 3 197 textes, interlettrage sur 1 408.
- Animations d'ouverture et de fermeture `Dlg_Anim_Open` / `Dlg_Anim_Close` sur 38 pages sur 82 ; les fenêtres démarrent
  dans l'état d'avant leur animation d'entrée. 22 clips ont des pistes d'opacité, de couleur ou de transformation 2D ;
  d'autres pilotent des paramètres de matériaux.
- Les sprites sont des PaperSprite ; les atlas (ex. `FWindow_Journal`) se résolvent par dépendances.
- Les textes sont des clés (`%UI_…`) vers une table chiffrée : aucun libellé lisible. Les pages contiennent des textes
  d'exemple du concepteur, et des éléments d'aperçu (recharges, boss, devises factices) qu'on ne peut pas filtrer par le
  seul mot « Dummy ».
- `FlowPanel` propre à AION 2 : enfants fixes sur une grille de `NumOfFixed` colonnes (4 par défaut), direction
  « RightAndUp ».
- `ListView`, `Carousel`, `DynamicEntryBox`, `FlowPanel` ne se peuplent qu'à l'exécution. `NamedSlotBindings` et les
  paramètres d'instance (ex. `OptionSlotList` : type de catalogue, taille, colonnes) doivent être lus dans l'export brut :
  les propriétés d'une instance de gabarit (icône, texte) ne figurent pas dans l'arbre du gabarit.

### Pièges de lecture (vérifiés)
- **Valeurs par défaut omises** : un enfant d'Overlay sans alignement écrit est en haut à gauche, un enfant de Button est
  centré. Une `Visibility` absente vaut le défaut natif (`SelfHitTestInvisible` pour CanvasPanel, Overlay, SafeZone) :
  la forcer à `Visible` fait intercepter les clics par des panneaux transparents.
- **Styles** : les `ColorSlider` rangent leurs poignées dans `Style`, pas dans `WidgetStyle` ; les brosses par défaut d'une
  jauge font 32 px de haut (les plaques de nom attendent 8 px).
- `Common_Empty_Sprite` est une vraie texture transparente : tester l'absence de ressource ne dit pas si un bouton a une
  icône.
- Une image sans matériau est en `NoDrawType` hérité, à remplacer pour l'afficher.
- Un chemin de bouton s'écrit sans barre finale (`FWindow_NPCTalk/P_QuestInfo/P_Btn_Accept`).
- `P_Img_BG` des écrans de création attend une cible de rendu.

### Police
- La fonte d'interface `Aion2Atreia_EN_Bold.ttf` ne contient pas les lettres accentuées françaises ni « © ».

## Supposé
- Les matériaux d'interface se reproduiraient fidèlement en relisant leurs shaders compilés (voir
  [shaders-compiles](shaders-compiles.md)) ; non fait.
- Les familles de polices réelles (`TypefaceFontName`) n'ont pas encore été relevées page par page.

## Pour reproduire
1. Exporter une page et toutes ses dépendances, en plusieurs passes jusqu'à ce qu'aucune nouvelle n'apparaisse.
2. Compter les enfants d'Overlay sans alignement écrit : ils doivent rester en haut à gauche.
3. Lire `UIScaleCurve` dans la configuration et tester en 1600 × 900 puis 2560 × 1440.

Pages et HUD : voir [interface-hud-fenetres](interface-hud-fenetres.md).
