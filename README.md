# AION 2 — étude technique partagée / shared technical research

**FR** — Carnet de recherche ouvert sur le fonctionnement technique d'AION 2 (Unreal Engine 5) : organisation du
contenu, formats, matériaux, rendu, animation. Chacun, avec ou sans IA, peut y apporter ses trouvailles, les vérifier et
en discuter. On partage de la **compréhension**, jamais le contenu du jeu.

**EN** — Open research notebook about how AION 2 works under the hood (Unreal Engine 5): content layout, formats,
materials, rendering, animation. Anyone — human or AI agent — can add findings, verify them and discuss them. We share
**understanding**, never game content. Notes are in French; English contributions are welcome.

## Règles / Rules

1. **Aucun contenu du jeu** : ni modèle, texture, son, animation, table de données ou texte extrait.
   *No game content: no extracted models, textures, sounds, animations, data tables or text.*
2. **Aucune clé ni outil de déchiffrement** des archives. Dire « lisible avec tel outil » suffit.
   *No decryption keys or archive-cracking tools.*
3. **Une trouvaille = une méthode vérifiable** : version du client, comment c'est observé, ce qui est établi et ce qui
   est supposé. *Every finding states client build, method, what is established and what is assumed.*
4. Pas de serveurs privés, pas de distribution, pas de triche. *No private servers, redistribution or cheats.*

## Comment participer / How to contribute

- **Une trouvaille** : une note Markdown dans le bon dossier (modèle : [`modele-trouvaille.md`](modele-trouvaille.md)), en pull request.
- **Une question, une piste, un désaccord** : l'onglet *Discussions*.
- **Une erreur dans une note** : une *Issue* qui cite la note et la preuve contraire.
- **Agents IA** : lire d'abord [`AGENTS.md`](AGENTS.md).

## Sommaire / Contents

**Lire le client**
- [Builds, mappage .usmap, tables lisibles ou chiffrées, pièges d'export](lecture-du-client.md)
- [Matériaux : lire le calcul dans les shaders compilés (DXIL)](shaders-compiles.md)

**Décor et rendu**
- [Textures en masques par canal](textures-masques.md)
- [Matériaux : interrupteurs à reproduire](materiaux-interrupteurs.md)
- [Matériaux : tuiles répétées (troncs)](materiaux-tuiles.md)
- [Matériau de paysage (EM_Landscape)](paysage.md)
- [Maillages : niveaux de détail, Nanite](maillages-nanite.md)
- [Ciel : étoiles, aurores, créatures du ciel](ciel.md)
- [Écran titre et salle de sélection](lobby-et-ecran-titre.md)
- [Éclairage propre aux personnages (CharLightsSetting)](eclairage-personnages.md)

**Personnage**
- [Création et personnalisation : tables, réglages, caméra](personnalisation-personnage.md)
- [Control Rig cuites : morphologie du corps et du visage](control-rig-morphologie.md)
- [Peau, maquillage, sourcils, yeux](peau-et-yeux.md)
- [Cheveux : coiffures et matériau CMM_Hair](cheveux.md)
- [Physique des cheveux, vêtements et ailes (KawaiiPhysics)](physique-cheveux-vetements.md)
- [Armures, tenues et armes](armures-et-tenues.md)
- [Ailes et vol](ailes-et-vol.md)

**Animation et actions**
- [Animations du personnage : saut, armes rangées, prises en main](animations-personnage.md)
- [Animations de combat : montages, esquives, banques par arme](animations-combat.md)
- [Acts : actions, effets, sons, cinématiques](acts-et-effets.md)
- [Émotes (SocialAction)](emotes.md)
- [Contrôles et caméra](controles-et-camera.md)

**Monde, interface, son**
- [Monstres, PNJ et baleine du vide](monstres-et-pnj.md)
- [PNJ immobiles et sons d'interaction](pnj-et-sons.md)
- [Interface UMG : organisation, échelle, pièges](interface-umg.md)
- [Interface : HUD et fenêtres principales](interface-hud-fenetres.md)
- [Sons et musique : organisation Wwise](sons-wwise.md)

Licence des notes : [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.fr). AION 2 est une marque de
NCSoft ; ce dépôt n'est ni affilié ni approuvé par NCSoft.
