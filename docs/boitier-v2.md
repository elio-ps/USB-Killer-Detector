# Boîtier et PCB v2 : vérification des découpes

`PCB/Case/Case.FCStd` et `Lid.FCStd` datent du 2 octobre 2026, donc d'avant le PCB v2.
Ce document compare leurs découpes avec la carte v2 et décrit les corrections apportées par
la macro `PCB/Case/maj_boitier_v2.FCMacro`.

> **État au 29 octobre 2026 :** les cotes ci-dessous ont été calculées à partir du PCB
> (KiCad 10.0.3), des esquisses lues dans les `.FCStd` et des plans constructeur. La macro a
> ensuite été **exécutée** (FreeCAD 1.1.4, `freecadcmd`) :
>
> ```
> Boitier : 11 fonctions V2 ajoutees
> Couvercle : 34 trous d'aeration supprimes
> Couvercle : 13 fonctions V2 ajoutees
> Controle : 75 pieces, 0 interference(s)
> ```
>
> Contrôle du nouveau `PCB/Model.3mf` par calcul : les deux maillages sont fermés, et 22 points
> de contrôle (fentes, ouverture, trous, tubes, encoche, aérations retirées ou conservées) tombent
> du bon côté, matière ou vide. Pas encore imprimé.

## Fichiers

| Fichier | Contenu |
|---|---|
| `PCB/SENTINELLE-v2-PCB.step` | Modèle 3D du PCB v2 exporté par `kicad-cli pcb export step --subst-models`. **J1 (USB1046) et F1 n'y figurent pas** : la bibliothèque KiCad 10 n'a pas leur modèle 3D. La macro ajoute une boîte aux cotes du plan GCT pour J1. |
| `PCB/Case/maj_boitier_v2.FCMacro` | Applique les corrections sur `Case.FCStd` et `Lid.FCStd`, contrôle les interférences avec le STEP et régénère `PCB/Model.3mf` |

## Repère

- Boîtier : 48 × 85 × 15 mm, poche de 45 × 83 mm, fond intérieur à z = 3 mm, parois d'extrémité de 1 mm.
- Le PCB (44 × 82 mm) est posé à plat sur le fond : dessous à z = 3, dessus à z = 4,6 mm.
- Passage KiCad → boîtier : `x = -(x_KiCad - 88,5)`, `y = y_KiCad - 99` (vue de dessus du boîtier, USB-A en +y).
- Couvercle : plaque de 2 mm, anneau de 4 mm, lèvre de 3 mm qui entre dans la poche. Fermé, le dessus
  du couvercle est à z = 21 mm et le bas de la lèvre à z = 12 mm.

## Résultat de la vérification

| Élément (PCB v2) | Position et encombrement dans le boîtier | Découpe actuelle | Verdict |
|---|---|---|---|
| **J1 USB-A** (GCT USB1046) | x −6,4…+6,4 ; z 4,6…11,72 (hauteur 7,12 ± 0,38) ; dépasse de 4,5 mm hors du boîtier, collerette de 14,5 mm à l'extérieur | Fenêtre fermée 14 × 5 mm, z 5…10 | ❌ Le corps touche en haut et en bas, et la collerette ne peut pas passer : la carte ne peut pas être montée |
| Lèvre du couvercle au-dessus de J1 | Haut de J1 à 11,72 mm (jusqu'à 12,1 mm avec la tolérance) | Bas de la lèvre à 12 mm | ⚠️ Contact possible |
| **J2 USB-C** (GCT USB4125) | Axe x = +0,82 ; coque x −3,65…+5,29, z 4,6…8,2 ; nez à 0,4 mm en retrait de la face extérieure | Fente 7,5 × 3 mm centrée en x = −0,06, z 5…8 | ❌ Décalée de 0,88 mm, trop étroite (8,94 mm nécessaires) et trop basse |
| **5 LED 5 mm** (D2 PWR, D9 D+, D10 D−, D11 VBUS, D15 OK) | D2 (16,73 ; −35,5), D9/D10/D11 (−13,0 ; 9,56 / 17,03 / 24,5), D15 (−13,23 ; −13,5) | Aucun trou, seulement la grille d'aération Ø 2 mm | ❌ Voyants invisibles |
| **SW1 RESET** (B3S-1000) | (16,2 ; −3,5), haut du bouton à 8,9 mm (hauteur 4,3 mm d'après le modèle 3D KiCad) | Aucun accès | ❌ |
| Broches traversantes (LED, J1, J2) | Dépassent sous le PCB | Fond plat | ⚠️ Le PCB ne repose pas à plat, ce qui fausse toutes les hauteurs ci-dessus |
| Contour du PCB | 44 × 82 mm dans 45 × 83 mm | — | ✅ 0,5 mm de jeu par côté, coins chanfreinés compatibles avec les congés de 2 mm |

## Corrections apportées par la macro

Toutes les nouvelles fonctions sont ajoutées à la fin de l'arbre, avec le préfixe `V2_`. Les
fonctions d'origine ne sont pas modifiées, sauf la grille d'aération du couvercle (`Sketch003`),
dont les trous trop proches des nouveaux tubes sont supprimés.

**Boîtier**
- `V2_FenteUSBA` : fente en U de 14 mm, de z = 4,2 mm jusqu'en haut de la paroi. La carte se pose
  verticalement, avec la collerette de J1 à l'extérieur.
- `V2_OuvertureUSBC` : ouverture de 12,6 × 6,6 mm centrée sur x = +0,82, de z = 3 à 9,6 mm. Elle laisse
  entrer le surmoulage de la fiche (12,35 × 6,5 mm au maximum selon la norme), ce qui garantit
  l'enfoncement complet de la fiche, puisque la prise est en retrait.
- `V2_Degagement_*` : poches de 1,5 mm sous les broches des LED, de J1 et de J2. Couper les pattes à 1,2 mm au plus.

**Couvercle**
- `V2_EncocheLevreUSBA` : la lèvre est retirée sur 15 mm au-dessus de J1.
- `V2_Tube_D*` et `V2_Trou_D*` : trou de Ø 5,3 mm et tube guide-lumière de Ø 7,2 mm, qui descend jusqu'à z = 14 mm.
  Une LED soudée à plat (sommet à 13,2 mm) éclaire le tube. Une LED soudée plus haut entre dans le tube, et sa
  collerette (Ø 5,8 mm) bute en bas du tube : le dôme dépasse alors d'environ 0,6 mm du dessus du couvercle.
- `V2_Tube_SW1` et `V2_Trou_SW1` : trou de Ø 3,2 mm et tube guide jusqu'à z = 11 mm, soit 2 mm au-dessus du
  bouton. On appuie avec un trombone ou une pointe de stylo, ce qui évite les RESET accidentels.

Les trous et l'encoche rendent le couvercle orientable dans un seul sens : on le retourne autour de son grand
axe, encoche côté USB-A.

## Lancer la macro

1. FreeCAD 1.1 ou plus récent → **Macro → Macros…** → choisir `PCB/Case/maj_boitier_v2.FCMacro` → **Exécuter**
   (ou, en console, depuis `PCB/Case/` : `freecadcmd maj_boitier_v2.FCMacro`).
2. Lire le rapport dans la vue rapport : nombre de fonctions ajoutées, trous d'aération supprimés et
   **interférences**. On attend « 0 interférence ».
3. La macro enregistre `Case.FCStd` et `Lid.FCStd`, puis réécrit `PCB/Model.3mf` (boîtier, et couvercle à
   60 mm, plaque sur le plateau). L'ancien fichier était un projet PrusaSlicer : les réglages
   d'impression qu'il contenait ne sont pas repris.

## Valider (à la main)

- [x] La macro passe sans erreur et annonce 0 interférence (29 octobre 2026).
- [ ] Contrôle visuel dans FreeCAD : la fente USB-A, l'ouverture USB-C, les 5 tubes et le tube RESET tombent
      en face des composants.
- [ ] Imprimer le boîtier et poser la carte nue (ou un gabarit papier à l'échelle 1) avant la commande.
- [ ] Mesurer la hauteur réelle de SW1 : la datasheet Omron n'a pas été vérifiée, seul le modèle 3D KiCad l'a été.
- [ ] À réception des cartes (LED soudées par JLC), vérifier que le couvercle ferme. Les tubes commencent à 9,4 mm
      au-dessus du PCB. Une LED posée à plat (8,6 à 8,7 mm de haut) reste sous le tube ; une LED surélevée entre
      dedans, tant que sa collerette reste sous 9,4 mm.

## Pistes non faites

- Le PCB n'a pas de trou de fixation : il est seulement tenu par les parois.
- Au-dessus de J1, il reste une ouverture de 14 × 3,3 mm entre le connecteur et l'anneau du couvercle. Une
  languette sur le couvercle pourrait la fermer.
