# Textures en masques par canal

Build du client observé : Steam 25767555 (octobre 2026). Méthode : lecture des archives avec CUE4Parse, examen des canaux un par un.

- `BG/Common/Sky/Sky_Star01` (4096 × 2048) n'est pas une image couleur.
  - R et B portent la poussière d'étoiles fine (identiques), G les étoiles brillantes avec halo, A est plein.
  - Lue comme une couleur, elle donne des traînées magenta (R+B) et des points verts (G).
  - **Établi** par l'histogramme et l'affichage canal par canal.
- `BG/Common/Sky/Texture/ET_CurtainAurora_M` : masque de rideau d'aurore (G = rideaux, R = fond magenta).
- `T_Mask` des arbres (EMM_Tree_Leaf) : R = découpe, B = occlusion. Ne pas remplir l'opacité avec B.
- `T_BaseColor` valant `ET_Default_Color` (blanc) signale que la couleur vient d'ailleurs : `T_Tile_BaseColor` pour les
  matériaux EMM_Object « à tuile » (troncs de `COM_TreeE01/E02`).
