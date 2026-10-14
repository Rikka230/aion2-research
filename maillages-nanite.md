# Maillages : niveaux de détail, Nanite, version de secours

- Les arbres communs (`BG/Common/Nature/Tree/Tree_ENV`) ont 4 niveaux de détail classiques : l'export classique est
  complet.
- Certains décors (`BG/SharingMap/DraconicCrisis/.../DRA_WhaleA01`) n'existent qu'en Nanite.
- Avec CUE4Parse 1.2.2.202610, la lecture des ressources Nanite après les LOD échoue (« Read size is bigger than
  remaining archive length ») : le format d'AION 2 diffère, à étudier.
- Le lecteur générique de StaticMesh échoue dans `SerializeInlineDataRepresentations` : il faut lire les LOD puis
  s'arrêter (lecteur sur mesure).
