# SENTINELLE — détecteur d'USB Killer

SENTINELLE est une **carte électronique**, pas un logiciel. On y branche un périphérique USB suspect, et elle
l'alimente comme le ferait un ordinateur. Pendant ce temps, elle surveille ses lignes **D+**, **D−** et
**VBUS**. Dès qu'une tension anormale apparaît, positive ou négative, un voyant rouge s'allume et **reste
allumé** jusqu'au RESET. Cette tension anormale est la signature d'un USB Killer : ce type de clé charge des
condensateurs sur le 5 V du port, puis les décharge en haute tension sur les lignes du port.

Le périphérique testé n'est relié à aucun ordinateur. La prise USB-C de la carte ne sert qu'à l'alimenter, et
il n'y a rien à installer.

> **État (octobre 2026) :** la v2 est conçue sous KiCad 10, et l'ERC comme le DRC sont propres. Elle n'est pas
> encore fabriquée, ni testée sur une vraie carte : les seuils ci-dessous sont calculés, pas mesurés.

## Utilisation

1. Alimenter la carte par la prise **USB-C** (chargeur 5 V ou port d'ordinateur) : le voyant orange **PWR**
   et le voyant vert **OK** s'allument.
2. Brancher le périphérique à tester sur la prise **USB-A**.
3. Lire les voyants. Un voyant rouge allumé signale une tension anormale sur la ligne correspondante, et le
   voyant vert s'éteint.
4. Appuyer sur **RESET** (ou débrancher l'alimentation) pour effacer l'alarme.

## Fonctionnement (blocs A à H du schéma)

Le schéma est `sch/usb-killer-detector.kicad_sch`, également disponible en PDF : `sch/fabrication/schema-v2.pdf`.

| Bloc | Rôle | Composants |
|---|---|---|
| **A. Alimentation USB-C (5 V)** | Prise d'alimentation sans lignes de données. Les résistances CC permettent à un chargeur USB-C de fournir le 5 V. Une diode TVS protège le 5 V | J2 (USB4125, 6 broches), R1/R2 5,1 kΩ, D1 SMAJ5.0A, C1/C2, voyant PWR D2 + R3 |
| **B. Port sous test USB-A et protection VBUS** | Fournit le 5 V au périphérique testé. D3 empêche toute tension de remonter vers l'alimentation, F1 (fusible réarmable de 500 mA) coupe en cas de surcharge, et D4 écrête VBUS | J1 (USB-A), D3 SS34, F1 MF-MSMF050-2, D4 SMAJ5.0A |
| **C. Références de seuil VH / VL** | Pont diviseur sur le 5 V : VH = 2,42 V et VL = 0,76 V. Comme ces références suivent le 5 V, les seuils aussi | R4 51 kΩ, R5 33 kΩ, R6 15 kΩ, C3/C4 |
| **D. Étages d'entrée (fenêtre ±)** | Chaque ligne est ramenée dans la plage 0–5 V par un pont à haute impédance. Exemple pour D+ : 2 × 220 kΩ en 1206 (R7 + R8), puis 220 kΩ vers le 5 V et 120 kΩ vers la masse, ce qui donne X = 0,15 × V + 1,5 V. Une BAT54S protège les comparateurs et un condensateur de 1 nF filtre le signal | D+ : R7–R10, C5, D5 ; D− : R11–R14, C6, D6 ; VBUS : R15–R17, C7, D7 |
| **E. Comparateurs LM339** | Deux comparateurs par ligne, l'un pour « trop haut » (X > VH), l'autre pour « trop bas » (X < VL). Leurs sorties à collecteur ouvert sont reliées par paires : l'alarme de chaque ligne est active à l'état bas | U1 (D+, D−), U2 (VBUS + commande du voyant OK ; le 8ᵉ comparateur est inutilisé, entrées fixées) |
| **F. Mémoire d'alarme 74HC279** | Trois bascules R/S. Une alarme, même très brève, fait basculer la sortie Q de la ligne à 1, et elle y reste jusqu'au reset | U3 |
| **G. Voyants** | Chaque sortie Q allume sa LED rouge. Un OU à diodes regroupe les trois alarmes et, par un comparateur de U2, éteint la LED verte | D9/D10/D11 + R22–R24, D12–D14, R25, D15 + R26 |
| **H. Découplage, pull-ups, reset** | 100 nF par circuit intégré et pull-ups des alarmes. Reset à la mise sous tension (R21/C11), décharge de C11 par D8 quand on débranche, bouton RESET | C8–C10, R18–R20, R21, C11, D8, SW1 |

Sur D+ et D−, les entrées présentent environ 440 kΩ : la carte n'absorbe donc pas l'énergie d'une décharge.
Sous −200 V, chaque résistance 1206 voit environ 100 V. Sur le PCB, ces lignes ont 1 mm d'isolation jusqu'aux
résistances d'entrée (règles dans `sch/usb-killer-detector.kicad_dru`).

## Seuils d'alarme

| Ligne | Alarme si | Valeurs calculées |
|---|---|---|
| D+ et D− | **> +6,2 V** ou **< −5 V** | +6,16 V / −4,95 V |
| VBUS | **> +6,3 V** ou **< −0,3 V** | +6,34 V / −0,31 V |

Une ligne USB 2.0 normale ne dépasse pas 3,6 V sur D+/D−, et VBUS reste entre 4,75 et 5,25 V. Le seuil VBUS se
règle avec R16/R17. Calculés avec des résistances à ±1 %, les seuils de D+/D− restent dans une fenêtre d'environ
±0,35 V (voir `docs/bom-jlcpcb.md`). Pour les mesurer sur une carte réelle, suivre
[docs/protocole-test.md](docs/protocole-test.md).

## Voyants

| Voyant | Couleur | Signification |
|---|---|---|
| PWR (D2) | orange | Carte alimentée |
| OK (D15) | vert | Aucune alarme mémorisée |
| D+ (D9) | rouge | Tension anormale vue sur D+ (mémorisée) |
| D− (D10) | rouge | Tension anormale vue sur D− (mémorisée) |
| VBUS (D11) | rouge | Tension anormale vue sur VBUS (mémorisée) |

## Reset

Une alarme reste affichée même si l'anomalie n'a duré qu'un instant : rien ne passe inaperçu en cas de
manque d'attention. Pour l'effacer :
- appuyer sur **RESET** (SW1) : une fois le boîtier mis à jour, on l'atteint avec un trombone par le trou du couvercle ;
- ou débrancher l'alimentation : au rebranchement, les alarmes repartent à zéro.

Si la cause est toujours présente au moment du RESET, l'alarme se rallume aussitôt.

## Fabrication (`sch/fabrication/`)

| Fichier | Contenu |
|---|---|
| `SENTINELLE-v2-gerbers.zip` | Gerber et perçages (contenu de `gerbers/`) : **à téléverser chez JLCPCB** |
| `SENTINELLE-BOM-JLCPCB.xlsx` | BOM au format JLCPCB avec les numéros LCSC vérifiés, LED comprises : **à téléverser** |
| `SENTINELLE-CPL-JLCPCB.csv` | Positions des composants au format JLCPCB : **à téléverser** |
| `BOM.csv`, `placement-CPL.csv` | Exports bruts de KiCad |
| `plan-assemblage-dessus.pdf` | Plan de placement, face composants |
| `schema-v2.pdf` | Schéma complet |
| `rapports/ERC.rpt`, `rapports/DRC.rpt` | Rapports de vérification de `kicad-cli` |

Carte : 2 couches, 44 × 82 mm, 1,6 mm d'épaisseur. **Tout est soudé par le fabricant**, y compris les 5 LED de
5 mm traversantes : il faut donc commander l'assemblage des composants traversants. Choix des références : `docs/bom-jlcpcb.md`.

## Boîtier (`PCB/`)

`PCB/Case/Case.FCStd` et `Lid.FCStd` (FreeCAD), `PCB/Model.3mf` pour l'impression, et
`PCB/SENTINELLE-v2-PCB.step` (modèle 3D de la carte). La vérification des découpes par rapport à la v2 est dans
`docs/boitier-v2.md`.

Le boîtier est à jour pour la v2 : fente USB-A, ouverture USB-C, 5 trous de LED avec tubes guide-lumière, trou
RESET et dégagements sous les broches. Il n'a pas encore été imprimé. Au montage, on retourne le couvercle autour
de son grand axe, encoche de la lèvre côté USB-A.

## Démo

> TODO : vidéo de la carte v2 en fonctionnement.

[`demo.gif`](demo.gif) (30 Mo) montre le **prototype sur breadboard** (v1), testé à l'alimentation de labo.

## Sécurité

Ne jamais brancher la prise USB-A de SENTINELLE sur un ordinateur. Ne pas toucher la carte pendant un test :
la haute tension d'un USB Killer est présente sur la prise USB-A et sur les résistances d'entrée. La validation
se fait sans vrai USB Killer, à l'alimentation de labo (voir le protocole de test).

## Équipe

- Elio Pironneau-Sauranus
- Lucas Rios
- Lenny Gaillard
- Evan Siby

Merci à **Philippe Mege**.

## Licence

GNU GPL v3.0 — voir [LICENSE](LICENSE).
