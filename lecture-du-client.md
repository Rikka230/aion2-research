# Lire le client : builds, mappage, tables, pièges d'export

- **Build du client** : Steam 25767555 (octobre 2026) et Steam 26077854 (21 octobre 2026)
- **Sujet** : archives `.pak` / IoStore d'AION 2 (Unreal Engine 5.3 cuit), fichier de mappage `.usmap`, tables `UnrealData/` et `Data/Table/`
- **Méthode** : CUE4Parse (.NET), export JSON et glTF, comparaison des index de chemins entre deux builds. Lecture seule des archives, jeu arrêté.

## Établi

### Deux builds comparés
- 25767555 : 1 066 archives, index de **762 151** chemins.
- 26077854 : 1 126 archives, index de **780 115** chemins ; **18 290** ajouts et **498** retraits par rapport au précédent.
  Le compte ne tombe pas juste (762 151 + 18 290 − 498 = 779 943) : 172 chemins restent à expliquer (doublons de casse ?).
- Principaux ajouts : `Map/Intersever/Abyss` (≈ 2 000), `FX/Particle/Monster` (≈ 1 700), `Map/Dungeon/Quest` (≈ 1 300),
  `BG/Common02/Halloween` (≈ 1 000), `Character/Player/GM` (≈ 640 : armures, casques, `Special`, animations),
  `FX/Act/Monster` et `Sound/Act/Monster`, `BG/SharingMap/Party`, `BG/Common02/Nature`, `GameFeatures/SeasonalEvent`,
  `Character/Player/GF`, `Act/Cinematic/Cut_Scene` ; des armes de toutes les familles, une aile `WIngZ_010` (casse d'origine),
  la coiffure `GM_Hair_416`, des textures d'interface et de carte du monde.
- Retraits notables : animations `*_Sidle_*_AM` (GF et GM), matériaux des ailes `WingZ_001`, ABP de pantalon de deux costumes.

### Mappage `.usmap`
- Les propriétés ne sont pas versionnées : sans `.usmap` de la bonne version, rien ne se lit.
- Le même `.usmap` (moteur 5.3.2, jeu 2.0.5.0) lit le build 26077854 sans erreur : exports JSON identiques sur 4 paquets
  témoins. Aucun `.usmap` public n'était disponible fin octobre 2026.
- Le fichier n'est pas compressé. Il donne les **noms exacts** des énumérations, structures et évènements (modes de
  contrôle, actions d'entrée, réglages), même quand les tables qui les utilisent sont chiffrées. Il ne donne aucune valeur.

### Ce qui se lit et ce qui ne se lit pas
- Lisibles : environ 140 DataTables et DataAssets de `UnrealData/` (costumes, personnalisation, caméras, aperçus, tables
  de ressources des PNJ…).
- Chiffrées par un procédé propre à l'éditeur, que la lecture des archives n'ouvre pas : `Data/Table/*.dat` (compétences,
  objets, textes `L10NString`, `InputAction`, `KeyGuide`, `SkillControlSetting`, `LobbyCreateCharItem`,
  `CustomizingPreviewArmors`…). Les valeurs par défaut des réglages et tous les textes manquent donc.
- Les vidéos `.mp4` des cinématiques sont hors des archives.
- Dépendances de décompression : Oodle pour les archives, ACL pour les animations compressées, Detex pour certaines
  textures (masques de l'œil). CUE4Parse les prend en charge si les bibliothèques natives sont présentes.

### Pièges d'export (vérifiés)
- **Valeurs par défaut omises** : l'export n'écrit pas une propriété égale au défaut de sa classe. Pour l'UMG, voir
  [interface-umg](interface-umg.md). Une propriété absente n'est donc pas « nulle ».
- **Réglages liés par PropertyAccess** : dans une AnimBP cuite, la valeur visible dans le nœud est un leurre ; la vraie
  source se suit par `SerializedSparseClassData.AnimBlueprintExtension_PropertyAccess.Library` (voir
  [physique-cheveux-vetements](physique-cheveux-vetements.md)).
- **Valeurs par défaut des matériaux** : dans `CachedExpressionData` du maître, pas dans l'instance.
- **Vecteurs à 4 composantes** : un import qui les tronque à 3 fausse certains paramètres (reflet de l'œil,
  `EyeHighLightTexOffset` = (1, 1, 0,024, 1)).
- **Casse variable** : `GM_0041_T04_pants`, `GM_0108_T04_shoulder`. Toute recherche dans l'index doit ignorer la casse.
- **Sprites** : certains fichiers commencent par un `BodySetup` ; ne lire que le premier objet en fait manquer (149 sur 428
  dans la carte du monde).
- **Maillages statiques** : le lecteur générique échoue après les LOD (voir [maillages-nanite](maillages-nanite.md)).
- **Animations exportées** : une animation additive sort avec ses clés déjà appliquées sur la pose de repos ; les
  quaternions de repos ne sont pas tous normalisés (norme ≈ 0,998), à normaliser avant de mesurer un angle.
- **Normales** : convention DirectX, souvent sans canal B (inverser G, reconstruire Z).
- **Armes `*_0000_T01`** (`SW_`, `MA_`, `MB_`, `DA_`, `ST_`, `OR_`) : maillages vides à l'export (bornes de 0,00002 cm,
  matériau `WorldGridMaterial`), alors que la série `0006_T04` est complète.
- Les chemins d'export longs dépassent la limite de 260 caractères de Windows : aplatir l'arborescence de sortie.

## Supposé
- Les armes `0000_T01` sont peut-être des maillages fictifs voulus (armes de départ remplacées à l'exécution) plutôt
  qu'un défaut de lecture ; non tranché.
- Une propriété non sérialisée vaut probablement le défaut de la classe C++, que les archives ne contiennent pas.

## Pour reproduire
1. Lister les chemins des deux builds et comparer les listes triées sans tenir compte de la casse.
2. Exporter en JSON le même paquet avec le même `.usmap` dans les deux builds et comparer.
3. Pour une AnimBP, chercher `PropertyAccess` dans le JSON et remonter jusqu'à la variable source.
