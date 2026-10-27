# Sons et musique : organisation Wwise

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : `WwiseAudio/Events/` (évènements), `WwiseAudio/Event/<n>` (banques numérotées), médias `.wem`
- **Méthode** : suivi des évènements Wwise jusqu'à leurs médias dans les archives, décodage avec vgmstream, comparaison
  des durées et des enveloppes

## Établi
- Tous les médias examinés sont des `.wem` (Vorbis Wwise ou PCM) et se décodent avec vgmstream sans échec.
- Les noms de médias sont en coréen, suffixés d'un CRC. Un même média apparaît dans plusieurs banques : les fichiers
  suffixés d'un numéro sont des doublons de banque. Seuls les fichiers marqués comme sources (« SFX ») sont distincts.
- Menu (titre, salle, création) : 80 évènements `UI_Interface_*` → 595 médias (≈ 79 Mo) : musiques, clics, choix de race,
  de classe, d'emplacement, de tenue (1 à 4), types de lumière (`UI_Interface_Customizing_LightType_Indoor` /
  `_Outdoor`), voix de classe.
- Musique : `WwiseAudio/Events/Music/` contient 557 évènements, ≈ 28 Go avec les doublons, **1 095 sources distinctes**.
- Chargement : `Play_Music_UI_Loading_Scene` (banque `WwiseAudio/Event/39`), quatre pistes PCM stéréo 44,1 kHz 16 bits :
  `Loop_pad_FIX` 53,33 s, `Voice_FIX` 53,33 s, `Voice_D` 70 s, `Voice_L_bpm48` 65 s.
- Joueur : séparation systématique **1P / 3P** (`PC_1P/…`, `PC_3P/…`, `PC_Vo_1P/…`, `PC_Vo_3P/…`). Les acts portent les
  deux évènements (`Event`, `Event_3P`, `bUse3PEvent`). Voir [emotes](emotes.md) et [acts-et-effets](acts-et-effets.md).
- Boucles : un évènement `_Stop` peut référencer le même média que la boucle ; il arrête la boucle au lieu de jouer un son
  de fin (voir [pnj-et-sons](pnj-et-sons.md)).
- Monstres : voix d'agression `M_*_Aggro_Vo`. Lobby : `SEQ_Cam_Ch_Leave_Door`, `SEQ_Cam_Ch_Leave_Whoosh`.

## Supposé
- Le graphe Wwise (choix aléatoire, gains, enchaînement des voix du chargement) n'est pas lisible depuis les seuls médias :
  ne pas superposer toutes les pistes, ni déduire une race d'un suffixe `_D` / `_L`.
- Le choix de la musique par page du menu n'est pas établi par les données.

## Pour reproduire
1. Exporter un évènement Wwise en JSON et suivre ses références de médias.
2. Décoder les `.wem` avec vgmstream ; regrouper par durée et empreinte pour éliminer les doublons de banque.
