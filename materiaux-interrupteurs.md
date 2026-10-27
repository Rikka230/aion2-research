# Matériaux : interrupteurs à reproduire

- `Use_RVTBlend` (EMM_Tree_Trunk, certains EMM_Object) : le pied de l'objet prend la couleur du terrain, lue dans une
  texture virtuelle (Runtime Virtual Texture) que le terrain remplit. Sans elle, les « contreforts » plats au pied des
  arbres (aulne `COM_AlderA01`) ressemblent à des erreurs de modélisation. **Établi** : les pans existent dans les
  niveaux de détail classiques, et le terrain d'AION 2 écrit dans une RVT.
- `Use_Dxt1Normal` : la vraie normale est `T_Normal_DXT1` ; `T_Normal` vaut alors la normale neutre.
- Maître `CM_Mob_Phantom` (`VEH_SkyWhale_02`) : graphe non disponible dans les archives cuites. Reconstruction
  supposée : translucide, plus opaque sur les bords, liserés cyan émissifs.

## Mise à jour (build 25767555, fin octobre 2026)
- `Use_Dxt1Normal` : vérifié sur 7 normales de l'écran titre et 69 matériaux `EMM_Object` / `EMM_Object_Od` de la salle
  de sélection ; `T_Normal` pointe alors sur `ET_Default_Normal` (neutre). Sans cette règle, les gravures disparaissent.
- `CM_Mob_Phantom` : le graphe reste absent, mais son calcul a été relu dans le shader compilé (émission divisée par
  l'adaptation de l'œil, liseré `DDE6F6`) : voir [monstres-et-pnj](monstres-et-pnj.md) et
  [shaders-compiles](shaders-compiles.md). La reconstruction « translucide, plus opaque sur les bords » est à abandonner :
  le corps est un matériau **masqué** (seuil 0,35) à deux faces.
- Les interrupteurs statiques d'une instance (ex. `Use_Fresnel_Invert`) se vérifient dans le code compilé.
- `BaseDay_Blend_FogColor` (`EMM_Simple_Fog`) : à 1, la brume tend vers `FogInscatteringColor` de l'`AionEnvironmentVolume`.
- Paramètre `Brightness` sur certaines plantes sans masque ARM : `MI_PlantAngelica` = 2,5 (couleur multipliée en linéaire).
- Les variantes de matériaux d'une carte passent par les `OverrideMaterials` des composants : lire la carte, pas seulement
  le maillage.
