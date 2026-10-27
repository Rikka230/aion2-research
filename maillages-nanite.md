# Maillages : niveaux de détail, Nanite, version de secours

- Les arbres communs (`BG/Common/Nature/Tree/Tree_ENV`) ont 4 niveaux de détail classiques : l'export classique est
  complet.
- Certains décors (`BG/SharingMap/DraconicCrisis/.../DRA_WhaleA01`) n'existent qu'en Nanite.
- Avec CUE4Parse 1.2.2.202610, la lecture des ressources Nanite après les LOD échoue (« Read size is bigger than
  remaining archive length ») : le format d'AION 2 diffère, à étudier.
- Le lecteur générique de StaticMesh échoue dans `SerializeInlineDataRepresentations` : il faut lire les LOD puis
  s'arrêter (lecteur sur mesure).

## Mise à jour (build 25767555, fin octobre 2026)
- Avec un lecteur sur mesure qui lit les LOD classiques puis tente les ressources Nanite, deux arbres communs donnent encore une
  erreur Nanite : `COM_AlderA01` (« Read size is bigger than remaining archive length ») et `COM_TreeE02` (« Arithmetic operation
  resulted in an overflow »). Leurs LOD classiques se lisent (4 niveaux ; `COM_AlderA01` : 13 040 sommets, 11 716 triangles au LOD 0).
- Avec le lecteur générique, ces mêmes maillages n'ont pas de données de rendu (`RenderData` nul) : un export générique, Nanite ou
  non, échoue (« Mesh has no LOD data »).
- Question ouverte : ces arbres ont-ils une géométrie Nanite plus fine que le LOD 0 ? Tant que le format Nanite n'est pas décodé,
  le LOD 0 est la meilleure géométrie disponible.

## Mise à jour (fin octobre 2026)
- Une option d'export Nanite ajoutée à un lecteur sur mesure n'a rien apporté : le format Nanite d'AION 2 reste non décodé.
  Pour `COM_TreeE02`, seul le maillage de repli sort (1 LOD, 4 754 sommets, facettes visibles).
- `DRA_WhaleA01` est une pièce de donjon statique en Nanite seulement, pas une créature du ciel.
- Les armes déclenchent à l'import un repli de matériau incompatible avec Nanite ; désactiver Nanite sur la copie supprime
  l'avertissement.
