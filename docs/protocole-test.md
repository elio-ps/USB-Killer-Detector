# Protocole de test de SENTINELLE v2, sans USB Killer

On valide la carte **sans jamais utiliser de vrai USB Killer**. On simule ses effets à basse tension avec une
alimentation de laboratoire et une résistance série : surtension, puis tension négative, sur D+, D− et VBUS.
Pour chaque essai, on vérifie que le bon voyant s'allume, que l'alarme reste mémorisée et qu'elle s'efface
au RESET comme à la coupure d'alimentation. On relève aussi les seuils réels pour les comparer aux seuils
calculés.

## 1. Règles de sécurité (à lire avant de commencer)

1. **Jamais de vrai USB Killer**, ni d'autre source de haute tension. Pendant ces essais, aucune tension ne
   dépasse **±10 V**.
2. **Ne jamais relier la carte à un ordinateur.** On l'alimente avec un **chargeur USB-C 5 V** ou une batterie
   externe, jamais avec le port d'un PC : une erreur de câblage sur VBUS ne doit pouvoir abîmer qu'un chargeur.
3. **Aucun périphérique sur la prise USB-A** : on n'y branche que la fiche de test décrite au § 3.
4. **Limitation de courant de l'alimentation de labo réglée avant chaque essai** : 20 mA pour D+ et D−,
   100 mA pour VBUS.
5. **Câbler hors tension** : sortie de l'alimentation coupée, consigne à 0 V. On ne change jamais un fil sous tension.
6. **La résistance série est obligatoire** : elle limite le courant en cas d'erreur de fil (court-circuit vers
   GND ou vers VBUS).
7. **Tension négative = fils de l'alimentation inversés** : le + va à GND, le − va à la ligne à travers la
   résistance. Repérer les fils avec du ruban adhésif de couleur avant de commencer.
8. **Condensateur de l'essai VBUS négatif** (§ 7.2) : il doit être **non polarisé** (électrolytique bipolaire
   « NP » ou céramique), car il se recharge à l'envers pendant le contact. Ne jamais dépasser sa tension
   nominale, et le décharger (court-circuit à travers la résistance de 1 Ω) avant de le ranger.
9. Faire ces essais en présence de l'enseignant et sur un plan de travail dégagé, sans objet métallique qui
   traîne sous la carte.

## 2. Matériel

| Matériel | Usage |
|---|---|
| Alimentation de labo 0–10 V **avec limitation de courant** | Source de surtension ou de tension négative |
| Multimètre (impédance d'entrée de 10 MΩ) | Mesure de la tension **sur la broche**, pas sur l'alimentation |
| Résistance de 10 kΩ, ¼ W | Résistance série pour D+ et D− |
| Résistance de 22 Ω, 1 W | Résistance série pour VBUS |
| Résistance de 1 Ω, 1 W + condensateur **non polarisé** de 100 µF / 25 V | Impulsion négative sur VBUS (§ 7.2) |
| Fiche USB-A mâle vers bornier (« breakout ») | Accès aux broches de la prise USB-A (J1) |
| Chargeur USB-C 5 V (ou batterie externe) | Alimentation de la carte |
| Oscilloscope (facultatif) | Voir l'impulsion du § 7.2 |

## 3. Brochage et points de mesure

Fiche USB-A branchée dans J1 : **1 = VBUS, 2 = D−, 3 = D+, 4 = GND**. Toutes les masses (alimentation de
labo, multimètre, oscilloscope) se prennent sur la broche 4.

| Point | Où le mesurer | Valeur attendue au repos |
|---|---|---|
| +5 V | pastille 1 de C1 | 4,75 à 5,25 V (selon le chargeur) |
| VH | pastille 1 de C3 | 2,42 V × (+5 V mesuré / 5) |
| VL | pastille 1 de C4 | 0,76 V × (+5 V mesuré / 5) |
| VBUS | broche 1 de J1 | environ 4,5 à 4,8 V (+5 V moins la chute de D3) |

## 4. Contrôle à vide

1. Brancher la fiche de test (rien n'est encore relié à ses bornes), puis le chargeur USB-C.
2. Attendu : **PWR orange allumé, OK vert allumé, les 3 rouges éteints**. Si un rouge est allumé, appuyer sur
   RESET. S'il reste allumé, arrêter et chercher le défaut avant d'aller plus loin.
3. Relever le +5 V, VH, VL et VBUS dans le tableau du § 9. Les seuils réels sont proportionnels au +5 V : on
   corrige donc les seuils théoriques avec la tension mesurée (`seuil corrigé = seuil théorique × +5 V mesuré / 5`).

## 5. Essais sur D+ (broche 3)

Câblage : sortie de l'alimentation → résistance de 10 kΩ → broche 3. Masse de l'alimentation sur la broche 4.
Multimètre entre la broche 3 et la broche 4. Limitation à 20 mA.

**5.1 Surtension**
1. Consigne à 0 V, puis allumer la sortie de l'alimentation. Monter par pas de 0,5 V jusqu'à 5,5 V, puis par
   pas de 0,05 V.
2. Noter la tension **lue sur la broche** au moment où **la LED rouge D+ (D9) s'allume**. Attendu : vers
   **+6,2 V** (fourchette calculée de +5,82 à +6,51 V). Au même instant, la LED verte OK doit s'éteindre, et
   les LED D− et VBUS rester éteintes.
3. Faire les essais de mémorisation du § 8 avec cette alarme.

**5.2 Tension négative**
1. Couper la sortie et inverser les fils : **+ de l'alimentation sur la broche 4**, − de l'alimentation →
   10 kΩ → broche 3.
2. RESET, puis monter de 0 V vers 7 V par pas de 0,05 V à partir de 4 V. La broche devient négative.
3. Noter la tension sur la broche quand **D9** s'allume. Attendu : vers **−5,0 V** (fourchette de −5,21 à
   −4,70 V).

La résistance de 10 kΩ ne fausse pas la mesure, puisqu'on lit la tension sur la broche : l'entrée de la carte
présente environ 520 kΩ.

## 6. Essais sur D− (broche 2)

Mêmes essais qu'au § 5, sur la broche 2. La LED attendue est la **rouge D− (D10)**, avec les mêmes seuils
(+6,2 V et −5,0 V).

## 7. Essais sur VBUS (broche 1)

La carte fournit elle-même environ 4,6 V sur VBUS, à travers D3 et F1. D4 (SMAJ5.0A) commence à conduire
entre 6,4 et 7 V.

**7.1 Surtension**
1. Câblage : sortie de l'alimentation → résistance de 22 Ω → broche 1, masse sur la broche 4. Limitation à 100 mA.
2. Consigne à **5,0 V** avant d'allumer la sortie (ainsi, aucun courant ne circule au départ), puis monter par
   pas de 0,05 V.
3. Noter la tension sur la broche quand **la LED rouge VBUS (D11) s'allume**. Attendu : vers **+6,3 V**
   (fourchette de +6,13 à +6,56 V). Au-delà de 6,4 V environ, le courant de l'alimentation augmente parce que
   D4 se met à conduire : c'est normal. Ne pas dépasser 7,5 V.

**7.2 Tension négative (impulsion)**

Avec une alimentation continue, on ne peut pas rendre VBUS négatif : la carte maintient VBUS à +4,6 V, et il
faudrait plus de 1 A pour faire déclencher F1. On imite donc ce que fait un USB Killer, c'est-à-dire la
décharge d'un condensateur, mais à 5 V seulement.
1. Charger le condensateur de 100 µF (non polarisé) à **5 V** avec l'alimentation, puis le débrancher. Repérer
   la borne reliée au + de l'alimentation : c'est elle qu'on appelle « + » ci-dessous.
2. RESET de la carte.
3. Relier la borne **+** du condensateur à la broche 4 (GND), puis toucher brièvement la broche 1 (VBUS) avec
   sa borne **−**, à travers la résistance de 1 Ω.
4. Attendu : **D11 s'allume**. D'après le calcul, un oscilloscope sur la broche 1 devrait montrer VBUS
   descendre vers −0,7 à −1 V pendant environ 100 µs. Le seuil calculé est de −0,3 V (fourchette de −0,42 à −0,20 V).
5. Si rien ne s'allume, recommencer avec le condensateur chargé à 8 V, puis 10 V (sans dépasser sa tension
   nominale), et noter la charge qui déclenche l'alarme.

> Cette méthode a été calculée, mais pas encore essayée : noter le résultat, même négatif.

## 8. Mémorisation, RESET et coupure d'alimentation

À faire au moins une fois par ligne, juste après un déclenchement :

| Étape | Action | Attendu |
|---|---|---|
| M1 | Ramener l'alimentation de labo à 0 V et débrancher la ligne | La LED rouge **reste allumée**, la verte reste éteinte |
| M2 | Appuyer sur RESET | La rouge s'éteint, la verte se rallume |
| M3 | Redéclencher l'alarme et la laisser active, puis appuyer sur RESET | La rouge **reste allumée** (ou se rallume dès qu'on relâche) : la cause est toujours là |
| M4 | Redéclencher, couper l'alimentation de labo, débrancher le chargeur USB-C 5 s, puis le rebrancher | Rouges éteintes, verte allumée |
| M5 | Déclencher deux lignes l'une après l'autre (D+, puis VBUS) | Les deux LED rouges restent allumées, et un seul RESET efface les deux |

## 9. Tableau des résultats

Carte n° : ____ Date : ____ Testé par : ____ Chargeur utilisé : ____

| Mesure au repos | Théorique | Mesuré |
|---|---|---|
| +5 V | 5,00 V | |
| VH | 2,42 V | |
| VL | 0,76 V | |
| VBUS | 4,5 à 4,8 V | |

| Essai | LED attendue | Seuil théorique | Fourchette (±1 %) | Seuil corrigé (× +5 V / 5) | Seuil mesuré sur la broche | Écart | M1 | M2 | M3 | M4 |
|---|---|---|---|---|---|---|---|---|---|---|
| D+ surtension | D9 rouge D+ | +6,16 V | +5,82 … +6,51 V | | | | | | | |
| D+ négatif | D9 rouge D+ | −4,95 V | −5,21 … −4,70 V | | | | | | | |
| D− surtension | D10 rouge D− | +6,16 V | +5,82 … +6,51 V | | | | | | | |
| D− négatif | D10 rouge D− | −4,95 V | −5,21 … −4,70 V | | | | | | | |
| VBUS surtension | D11 rouge VBUS | +6,34 V | +6,13 … +6,56 V | | | | | | | |
| VBUS négatif (impulsion) | D11 rouge VBUS | −0,31 V | −0,42 … −0,20 V | — | charge du condensateur : | | | | | |

Essai M5 (deux alarmes, un seul RESET) : ☐ OK ☐ Pas OK

Remarques :

## 10. Si un résultat sort de la fourchette

- **Toutes les mesures décalées dans le même sens** : comparer d'abord le +5 V, VH et VL aux valeurs attendues.
- **Une seule ligne fausse** : vérifier les résistances de son étage d'entrée (D+ : R7–R10 ; D− : R11–R14 ;
  VBUS : R15–R17) et la BAT54S correspondante (D5, D6 ou D7).
- **La LED s'allume mais l'alarme ne reste pas mémorisée** : vérifier U3 (74HC279), et que RST_N est à +5 V
  au repos (C11, R21).
- **La LED verte ne s'éteint pas** : vérifier D12–D14, R25 et U2 (broches 10, 11 et 13).
- Seuil VBUS à changer : modifier R16/R17 (voir le README).
