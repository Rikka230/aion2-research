# Éclairage propre aux personnages (CharLightsSetting)

- **Build du client** : Steam 25767555 (octobre 2026)
- **Sujet** : réglages d'environnement d'AION 2 (1 460 dans l'index), bloc `CharLightsSetting`, courbes
  `ENV_Global_*LightCurve`, paramètre `CharacterEmissive` ; réglages relevés à Verteron, à Zumion et dans les aperçus
  `UnrealData/Environment/Setting/Preview_Customizing01`
- **Méthode** : export JSON des réglages d'environnement (CUE4Parse) et lecture des blocs de lumière

## Établi
- Chaque réglage d'environnement contient trois `RectLight` (principale, appoint, contre-jour), placées autour du
  personnage selon l'orientation de la caméra : décalage de tangage et de lacet, longueur de bras.
- Elles sont sur le **canal d'éclairage 2 seulement** (`bChannel0` = faux, `bChannel1` = faux, `bChannel2` = vrai) :
  le décor ne les reçoit pas.

| Moment (Verteron) | Principale | Appoint | Contre-jour |
|---|---|---|---|
| Aube | 1000, tangage −10°, lacet −40°, bras 191,7, `FFF7E3`, portée 700 | 800, lacet 23°, `FFF6EA`, portée 1000 | 800, tangage −30°, lacet 160°, bras 222,9, `FFF8ED`, portée 800 |
| Midi | 1100, bras 200, blanc, source 40 × 300 | 700, lacet 23° | 800, `FFDFB6`, portée 1000 |
| Soir | 900 | 500, lacet 10° | 800, `FFDFB6` |
| Minuit | 800, `CAF6FF` | 600, lacet 23,4° | 300, `FFF9F6` |
| Minuit, pluie | 800 | 600 | 800, `A4D7FF` |
| Zumion, minuit | 1300, lacet −30°, portée 800 | 1000, lacet 10° | 300 |

- La principale garde presque la même force à minuit qu'à midi (800 contre 1100) : la nuit, les personnages gardent
  leur modelé pendant que le décor s'assombrit.
- `UseAttenuationFromZoom` : la principale passe de 1 à 0,3 à mi-recul de la caméra ; appoint, contre-jour et spot de
  1 à 0 au tiers du recul (temps 0 = caméra au plus près).
- `CharacterEmissive` : 0 le jour, 0,01 à minuit à Verteron, 0,5 sous la pluie de minuit, 1,0 à Zumion.
- `LumenReflectionIntensityCharacter` 1,0 (0,3 sous la pluie) ; `AmbientOcclusionRadiusCharacter` 15 ;
  `CharacterBright`, `CharacterSkinBright`, `CharacterPlateBright` à 0.
- Une quatrième lumière (`SpotLight`, tangage −20°, lacet −40°, bras 137) n'a d'intensité (900) que dans la variante
  sans Lumen.
- Les aperçus de la personnalisation utilisent le même bloc (types de lumière intérieure et extérieure, voir
  [personnalisation-personnage](personnalisation-personnage.md)).

## Supposé
- Quand le bras n'est pas écrit dans l'export, il vaudrait le défaut de la structure (200 et 222,9 relevés ailleurs).
- L'unité du « recul » des courbes par rapport à la distance réelle de la caméra n'est pas établie.

## Pour reproduire
1. Exporter un réglage d'environnement d'une région et lire `CharLightsSetting`.
2. Dans Unreal 5 : trois `RectLight` sur le canal 2 seulement, attachées au personnage et orientées selon le lacet de la
   caméra ; les personnages doivent recevoir les canaux 0 et 2. Lumen ignore les canaux d'éclairage pour l'indirect :
   mettre l'intensité indirecte de ces lumières à 0.
