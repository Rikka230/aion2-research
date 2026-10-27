# Matériaux : lire le calcul dans les shaders compilés

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : matériaux maîtres sans graphe dans le jeu cuit (`CMM_Hair`, `CMM_Head`, `CM_Head_Cstm`, `CMM_Body`,
  `CM_Armor`, `CMM_Mob`, `CM_Mob_Phantom`, `EMM_AteriaCloud`, `EMM_AteriaRing`, `M_SmoothStep_Erosion_Addtive_Time_v1`…)
- **Méthode** : extraction du bytecode DXIL des archives, désassemblage avec `dxc -dumpbin` (SDK Windows), relecture
  des expressions qui alimentent chaque sortie du pixel shader.

## Établi
- Le jeu cuit ne contient pas les expressions des matériaux maîtres, mais il contient leur **code compilé**. Une
  reconstruction « à l'œil » peut donc être remplacée par le calcul réel.
- CUE4Parse ne lit pas la carte de shaders intégrée aux paquets d'AION 2. On retrouve le code en cherchant l'empreinte de
  la carte de shaders dans les octets du paquet, puis le groupe correspondant dans les archives IoStore (groupes compressés
  en Oodle).
- `dxc -dumpbin` donne un DXIL lisible ; en remontant depuis les sorties (couleur de base, normale, rugosité, émissif,
  opacité, données personnalisées), on obtient les formules. Les **interrupteurs statiques** d'une instance (ex.
  `Use_Fresnel_Invert`) se lisent aussi dans le code compilé : chaque permutation a son propre shader.
- La correspondance entre les emplacements de constantes et les noms de paramètres passe par une table de préshader au
  format propre à AION 2, non décodée. On l'établit en croisant la structure du calcul et les valeurs connues de
  l'instance.
- Plusieurs matériaux émissifs **divisent par l'adaptation de l'œil** (exposition) : leur luminosité à l'écran reste la
  même quelle que soit l'exposition. Exemple : la baleine `NPC_Void_001` (voir [monstres-et-pnj](monstres-et-pnj.md)).

Formules relevées par cette méthode, détaillées dans les notes thématiques :
- cheveux, peau, maquillage, paillettes, sourcils, œil : [cheveux](cheveux.md), [peau-et-yeux](peau-et-yeux.md) ;
- armures : [armures-et-tenues](armures-et-tenues.md) ;
- monstres et fantôme : [monstres-et-pnj](monstres-et-pnj.md) ;
- nuages et anneaux de l'écran titre : [lobby-et-ecran-titre](lobby-et-ecran-titre.md).

## Supposé
- Les noms des constantes déduits par recoupement peuvent être inversés quand deux paramètres ont la même valeur dans
  l'instance ; seule une lecture de la table de préshader le garantirait.
- Les matériaux d'interface (584 dans les pages exportées) se liraient de la même façon ; non fait.

## Pour reproduire
1. Repérer, dans les octets du paquet du matériau, l'empreinte de sa carte de shaders.
2. Retrouver le groupe de code dans les archives IoStore et le décompresser.
3. Désassembler chaque entrée DXIL avec `dxc -dumpbin` et suivre les registres jusqu'aux sorties.
4. Vérifier la formule en la reproduisant dans un matériau Unreal 5 et en comparant à une capture du jeu.
