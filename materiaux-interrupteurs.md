# Matériaux : interrupteurs à reproduire

- `Use_RVTBlend` (EMM_Tree_Trunk, certains EMM_Object) : le pied de l'objet prend la couleur du terrain, lue dans une
  texture virtuelle (Runtime Virtual Texture) que le terrain remplit. Sans elle, les « contreforts » plats au pied des
  arbres (aulne `COM_AlderA01`) ressemblent à des erreurs de modélisation. **Établi** : les pans existent dans les
  niveaux de détail classiques, et le terrain d'AION 2 écrit dans une RVT.
- `Use_Dxt1Normal` : la vraie normale est `T_Normal_DXT1` ; `T_Normal` vaut alors la normale neutre.
- Maître `CM_Mob_Phantom` (`VEH_SkyWhale_02`) : graphe non disponible dans les archives cuites. Reconstruction
  supposée : translucide, plus opaque sur les bords, liserés cyan émissifs.
