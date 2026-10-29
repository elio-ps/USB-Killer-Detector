# BOM JLCPCB : choix des références

Fichiers : `sch/fabrication/SENTINELLE-BOM-JLCPCB.xlsx` (onglet « BOM JLCPCB » à téléverser, onglet
« Details » pour les justifications) et `sch/fabrication/SENTINELLE-CPL-JLCPCB.csv`.

Chaque numéro LCSC a été vérifié le 29 octobre 2026 sur le catalogue JLCPCB : il existe, il est en stock et
son boîtier correspond. Le stock relevé ce jour-là figure dans l'onglet « Details ». **Tout est soudé par JLCPCB**,
y compris les 5 LED traversantes : l'équipe ne soude rien.

## R7, R8, R11, R12 : 220 kΩ 1206 à 1 %

**Rôle.** Dans les étages D (bloc D du schéma), chaque ligne D+ ou D− passe par deux résistances de 220 kΩ en
série (R7 + R8 pour D+, R11 + R12 pour D−) avant le point de mesure X. Ce point est aussi tiré vers +5 V par
R9/R13 (220 kΩ) et vers la masse par R10/R14 (120 kΩ) :

```
X = 0,15 × V_ligne + 1,5 V      (5 V d'alimentation)
alarme si X > VH = 2,42 V  →  V_ligne > +6,16 V
alarme si X < VL = 0,76 V  →  V_ligne < −4,95 V
```

Ces deux résistances fixent le gain du pont (0,15) et encaissent presque toute la haute tension : pour
−200 V sur la ligne, chacune voit environ 100 V et dissipe environ 45 mW, puisque X est bloqué près de 0 V
par la BAT54S.

**Effet de la tolérance** (pire cas, toutes les autres résistances à ±1 %) :

| R7, R8, R11, R12 | Seuil haut | Seuil bas |
|---|---|---|
| ±5 % (FOJAN proposée par JLC) | +5,68 … +6,67 V | −5,44 … −4,47 V |
| ±1 % | +5,82 … +6,51 V | −5,21 … −4,70 V |

**Choix : C873512, YAGEO RV1206FR-07220KL.** Il s'agit d'une 1206 « haute tension » : 1 %, 500 V par
résistance, 3 563 en stock. Avec deux en série, elle laisse plus de marge que la règle « ≥ 200 V ».
Repli si le stock baisse : **C17956**, UNI-ROYAL 1206W4F2203T5E (1 %, 200 V, 110 000 en stock). Aucune
220 kΩ 1206 n'existe en « basic » chez JLC : l'une comme l'autre compte comme pièce extended.

## Connecteurs, fusible, bouton

| Repère | Référence du schéma | LCSC | Remarque |
|---|---|---|---|
| J2 | GCT USB4125-GF-A | **C3151650** | Référence exacte. Ne pas prendre `-0190` (C5246813) : ses pattes plus longues correspondent à une autre empreinte KiCad |
| J1 | GCT USB1046 | **C6307429** (USB1046-GF-0190-L-B-A) | Référence exacte, mais **105 en stock seulement**, et GCT la marque « Not recommended for new designs ». À remplacer lors d'une prochaine révision du PCB |
| F1 | Bourns MF-MSMF050-2 | **C17313** | Référence exacte |
| SW1 | Omron B3S-1000 | **C2733655** | Le B3S-1000 CMS existe chez JLC (hauteur 4,3 mm). Il remplace le TL1220 traversant qu'avait proposé JLC : SW1 est donc soudé par JLC |

## LED de 5 mm (traversantes, soudées par JLC)

| Repère | LCSC | Référence | Vérifié sur la datasheet |
|---|---|---|---|
| D2 (PWR, orange) | **C52034657** | Guiguang 5A4OUD | Ø 5 mm, entraxe 2,54 mm, 8,7 mm de haut, 400–600 mcd à 20 mA |
| D9, D10, D11 (rouges) | **C87271** | Everlight 333-2SURD/S530-A3 | Ø 5 mm, entraxe 2,54 mm, 8,6 mm de haut, 200 mcd à 20 mA |
| D15 (OK, verte) | **C2895548** | Xinglight XL-502UGD | Ø 5 mm, entraxe 2,54 mm, 8,7 mm de haut, 4 à 8 cd à 20 mA |

- Les voyants tournent à environ 2 à 3 mA (résistances de 1 kΩ). D'où le choix de LED à lentille colorée assez
  lumineuses, en particulier la verte.
- Comme ce sont des composants traversants, la commande doit inclure **l'assemblage des composants traversants** chez
  JLC. C'est une option payante, qui n'est pas forcément proposée dans l'offre d'assemblage la moins chère : à
  vérifier sur le devis.
- **Polarité** : sur l'empreinte KiCad, la pastille carrée (1) est la cathode. Il faut contrôler dans l'aperçu JLC que
  le méplat ou la patte courte de chaque LED tombe bien sur cette pastille.
- **Hauteur** : le boîtier convient à une LED posée à plat comme à une LED surélevée jusqu'à environ 5 mm (le tube du
  couvercle commence à 9,4 mm au-dessus du PCB).

## Autres points

- **U3 (74HC279)** : le seul 74HC279 en SOIC-16 étroit disponible chez JLC est un clone, **C41430932**
  (Lingxingic SN74HC279ADR, 318 en stock). D'après sa datasheet, il a un boîtier de 3,9 mm au pas de 1,27 et le
  brochage 279 standard, vérifié broche par broche contre la netlist de U3. Les 74LS279 trouvés ne
  conviennent pas : ce sont des TTL, qui ne fournissent pas assez de courant aux LED.
- **Pièces extended** : 14 lignes, dont 2 « preferred » (R10/R14 et R17). Chez JLC, chaque pièce extended
  ajoute des frais de chargement. Le montant dépend de l'offre choisie : à vérifier sur le devis.
- **CPL** : c'est `placement-CPL.csv` avec les en-têtes JLC (`Designator, Mid X, Mid Y, Layer, Rotation`) et les
  coordonnées en mm. Les rotations sont celles de KiCad. Il faut **contrôler l'aperçu de placement JLC** (souvent
  à corriger : SOT-23, SOIC, diodes SMA et SOD-123, connecteurs USB), et corriger dans l'interface JLC si
  besoin.
