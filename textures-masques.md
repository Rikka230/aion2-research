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
- `T_Mask` des objets opaques (EMM_Object avec l'interrupteur `Use_ARM`) : R = occlusion, G = rugosité, B = métal.
  - **Établi** sur trois roches et une falaise de `BG/Common/Nature/Stone` : R sombre dans les creux (moyenne ~0,77),
    G entre 0,6 et 0,9 (moyenne ~0,65), B nul partout.
  - **Supposé** pour B : nul sur la pierre, donc compatible avec un canal métal, mais pas encore vérifié sur un objet
    métallique. Une observation contraire a été faite sur une colonne de salle (R qui suit le métal) : à départager sur
    un objet en métal du même maître.
  - Ne pas lire R comme une découpe sur ces matériaux opaques : la découpe en R ne vaut que pour les feuillages masqués.
