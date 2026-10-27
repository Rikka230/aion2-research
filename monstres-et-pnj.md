# Monstres, PNJ et baleine du vide : matériaux, tailles, animations

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : maître `CMM_Mob` ; `NPC_Void_001` (Blueprint `BPCh_NewVoid`), `CM_Mob_Phantom`,
  `M_SmoothStep_Erosion_Addtive_Time_v1`, `Void_01_AliveFx_Fx` ; `VEH_SkyWhale_02` ; PNJ de `Character/NPC/`
- **Méthode** : export JSON des Blueprints, instances et acts (CUE4Parse), export glTF ; shaders compilés (voir
  [shaders-compiles](shaders-compiles.md))

## Établi

### Matériau des monstres (`CMM_Mob`)
- Couleur = `_DO` / `_D` × teinture `_DYE` × 2 (`T_Gray74` = neutre). Les textures `_DO` sont souvent un détail gris :
  la couleur vient de `_DYE`. Alpha = découpe à 0,3333 ; normale ; ARM.
- Masque `_SEE` : R = diffusion (grandes zones), G = émissif (yeux, bandes, cartes de crinière). Paramètre
  `EmissiveDay_Color`. Exemple de jeu complet : `MOB_KrallWar_02` (DO, DYE, N, ARM, SEE).
- Variantes de couleur nommées `CV01`, `CV04`, `V01`…
- Jeu d'animations type par monstre (10 à 11) : attente, attente de combat, marche, course, `Atk01`, `Skill01`,
  `Skill02`, coup reçu, mort, mort au sol, et parfois un travail ; montages d'attaque et de coup reçu. Orientation de
  liaison à −90° en lacet.
- Hauteurs relevées : Worg ≈ 2,4 m, Airon ≈ 3,3 m.

### Baleine du vide `NPC_Void_001`
- Blueprint `BPCh_NewVoid` ; le maillage mesure environ **864 m**. Cycle « Run » de 16,7 s.
- Corps sur `CM_Mob_Phantom` (maître `CMM_Mob`) : matériau masqué (seuil 0,35), éclairé, à deux faces. Émission =
  mélange à 50 % d'un terme « slime » et d'un terme « fantôme », **divisé par l'adaptation de l'œil** (aspect blanc
  lumineux constant). Fantôme = couleur × 2, mêlée à un liseré `DDE6F6` (puissance 10), plus 22 % de la luminance du
  slime. Complète [materiaux-interrupteurs](materiaux-interrupteurs.md).
- Auréole : anneau fixe et horizontal, environ 30 m en avant et au-dessus du dos ; échelle de l'effet 8 × 8 × 20.
  Matériau `M_SmoothStep_Erosion_Addtive_Time_v1`, additif : motif `Pattern_205` qui défile, perturbé par `Noise_044` ;
  base bleue × 100, éclats roses × 500 au-dessus de 0,6, bord bleu × 50 sous 0,7 ; opacité = érosion × masque
  `Ramp_020_02` tourné de 270° × |N·V|^1,5 (`Use_Fresnel_Invert`) × 1,5.
- `Void_01_AliveFx_Fx` lance **six** systèmes : `Cast_001` (lueur et étincelles au centre de l'anneau), `Cast_002` et
  `Cast_006` (étincelles et lueurs émises sur la surface du corps), `Cast_003` (l'anneau et un cône), `Cast_004`
  (traînée), `Cast_005` (plan à 80 m au-dessus).
- Un matériau translucide sur ce maillage à deux faces mélange l'ordre des faces : le jeu utilise une découpe.

### `VEH_SkyWhale_02` est une monture
- `VEH_SkyWhale_02_SignatureFX` lance trois rubans (`Streak`) le long des os de la queue et des nageoires ; les effets
  `Wing`, `Haed`, `Morph` sont des étincelles (`MI_WingFx_Twinkel_002`). Aucune baleine de décor animée n'existe dans
  l'index. `DRA_WhaleA01` est une pièce de donjon statique, en Nanite seulement. Complète [ciel](ciel.md).

### PNJ
- Sur 62 PNJ examinés : 51 modulaires sur le squelette de joueur (GF / GM), 11 maillages complets (Shugo, Élims,
  personnes âgées, enfant, mannequin). Pour un PNJ modulaire, le « maillage complet » est la tenue entière
  (ex. `M_Comb_01`).
- Les PNJ n'ont pas d'armes propres : les tables renvoient à des armes de joueur (`Character/Player/Weapon`). Les objets
  des gestes (pelle `VTR_ToolA07`, corbeille `VTR_BasketA03`, lit `Weapon/Prop/BedA02`) sont posés par les acts
  (`AionActTask_Snapshot`, voir [acts-et-effets](acts-et-effets.md)).
- Attentes : `Stand_L_01` (bras croisés), `NPC_G*_Idle`, `NPC_G*_BW_Idle`, `NPC_G*_ST_Idle`. Les attentes `_AD_`
  (ex. `G?_AD_Farming_L_01`) animent l'os d'outil `Prop_R` = `DaggerWeapon`.
- `NPC_Elim_01` : Idle, Cidle, Stun (2 s), KnockDown (`KnockDownF_Loop`), Die ; pas de sommeil.
- Odar : voir [pnj-et-sons](pnj-et-sons.md).

## Supposé
- La correspondance des constantes de l'auréole repose sur la structure du calcul et les valeurs de l'instance (table de
  préshader non décodée).
- Le terme « slime » de `CM_Mob_Phantom` n'est reproduit qu'approximativement.
- Les émetteurs des six systèmes du Void n'ont pas encore été analysés un par un.

## Pour reproduire
1. Exporter `BPCh_NewVoid` et relever composants, transformations et l'act `AliveFx`.
2. Désassembler le pixel shader de `CM_Mob_Phantom` et chercher la division par l'exposition.
3. Pour un monstre, afficher séparément `_DYE` et les canaux de `_SEE`.
