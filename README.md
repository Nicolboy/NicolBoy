# Nicolas Boyer

<p align="center"><img src="doc/pwl.png" width="280" alt="PWL low noise microphone power"></p>

Power electronics and embedded systems — from datasheet to routed board,
from register-level firmware to LLM-driven agents in the hardware design
loop itself.

- **11–25V → 500V** — dual-stage boost HV supply, hardware trip-zone
  protection reacting in ~30ns, fully independent from the controlling MCU
- **100% identical connector** across two different C2000 MCU packages,
  generated from a single Markdown pin spec rather than hand-drawn in KiCad
- Natural-language drone control: a Jetson-hosted LLM turns French
  voice/text instructions into MAVLink commands, with real-time YOLOv8
  detection over a long-range video downlink

---

## Ce que je fais

**Conception électronique** — lecture et interprétation de schémas, choix de
composants sur datasheet, analogique et numérique. Chaînes de mesure de
courant, conditionnement, références de tension, protections.

**PCB** — conception et routage sous KiCad, du symbole au Gerber. Contraintes
de fabrication assumées : les projets existent en variante 2 couches gravée à
la maison pour itérer vite, et 4 couches en fabrication externe pour les plans
de masse continus.

**Conversion d'énergie** — boost, polyphasé, soft start, régulation numérique,
protections matérielles à déclenchement rapide. De quelques milliampères sous
48 V à 500 V sur batterie.

**Embarqué** — C sur MSP430, MSPM0, C2000, ESP32 ; Python sur Linux embarqué.
Écriture directe de registres quand la couche d'abstraction coûte trop cher, et
étalonnage fondé sur la mesure plutôt que sur le calcul.

**IA appliquée** — inférence embarquée sur Jetson, interprétation d'instructions
en langage naturel vers des commandes machine. Et l'usage d'agents dans la
conception matérielle elle-même : lecture de documentation technique, génération
de symboles, vérifications exécutables.

---

Électronique de puissance, embarqué, et ce qui va avec — de la lecture de
datasheet au routage, du registre au modèle de langage.

Les dépôts ci-dessous sont des projets réels, dans l'état où ils sont. Ce qui
ne fonctionne pas encore y est écrit.

---

## Deux projets en cours de réalisation

### Alimentation haute tension pour préamplis à tubes

Six dépôts, une seule chaîne : les deux cartes de puissance, la carte de
contrôle, la carte d'alimentation, le firmware qui régule, et le
superviseur qui pilote.

| | |
|---|---|
| [**KICAD-C2000-float-flyback**](https://github.com/Nicolboy/KICAD-C2000-float-flyback) | alimentation filaments (HT), carte KiCad 10 |
| [**KICAD-HV**](https://github.com/Nicolboy/KICAD-HV) | alimentation tension d'anode (200–500 V, boost double étage), carte KiCad 10 |
| [**KICAD-C2000-devkit**](https://github.com/Nicolboy/KICAD-C2000-devkit) | shield d'isolation et cartes devkit, KiCad 10 |
| [**KICAD-C2000-powershield**](https://github.com/Nicolboy/KICAD-C2000-powershield) | carte d'alimentation USB-C PD, liaison devkit/cartes filles, KiCad 10 |
| [**CCS-C2000-hv-boost**](https://github.com/Nicolboy/CCS-C2000-hv-boost) | firmware TMS320F28027 — deux étages boost, 10 V → 500 V |
| [**VSCODE-ESP32-hv-boost**](https://github.com/Nicolboy/VSCODE-ESP32-hv-boost) | ESP32-C3 — pont série, IHM OLED, API HTTP |

Deux étages de tension, deux logiques différentes.

<p align="center"><img src="doc/KICAD-HT-pcbway.png" width="280" alt="Carte HT, chauffage filaments"></p>

**Chauffage** : un étage abaisseur à découpage (buck), dont la tension de
polarisation permet d'élever le secondaire flottant jusqu'à 80 V.

<p align="center"><img src="doc/KICAD-HV.png" width="280" alt="Carte HV, tension d'anode"></p>

**Tension d'anode** : deux étages élévateurs (boost) en cascade, 200 kHz
puis 100 kHz, permettent d'obtenir entre 200 V et 500 V à partir d'une
tension d'alimentation entre 11 V et 25 V. La protection est **matérielle
et locale** via une logique rapide dédiée : comparateurs analogiques et
Trip Zone, environ 30 ns de réaction.

<p align="center"><img src="doc/devkit_B_F28P551.png" width="175" alt="Devkit F28P551"></p>

**Carte de contrôle** : cette carte de contrôle PWM/ADC est dessinée pour
deux microcontrôleurs différents — F280037 et F28P551 — avec un connecteur
**identique à 100 %**. Son brochage est décrit en Markdown, les symboles
KiCad en sont générés : les règles de conception deviennent exécutables au
lieu d'être seulement écrites. La boucle de régulation est intégralement
logicielle, dans le firmware de ces microcontrôleurs de la famille C2000
Texas Instruments, spécialisés dans la conversion d'énergie.

<p align="center"><img src="doc/shield.png" width="280" alt="Carte d'alimentation, liaison devkit / cartes filles"></p>

**Carte d'alimentation** : cette carte utilise une source d'alimentation
USB-C Power Delivery contrôlée de 100 W. Elle permet de connecter les
cartes filles de conversion à découpage au microcontrôleur et de les
alimenter. Elle porte aussi la carte ESP32 et un écran OLED pour la
visualisation et le paramétrage. Une page HTML accessible via hotspot
Wi-Fi permet de contrôler les tensions et de visualiser les états et
paramètres des cartes filles.

### WireClaw — drone d'inspection piloté en langage naturel

| | |
|---|---|
| [**VSCODE-JETSON-wireclaw**](https://github.com/Nicolboy/VSCODE-JETSON-wireclaw) | station sol Jetson Orin Nano — LLM → MAVLink, inférence YOLOv8 |
| [**VSCODE-PIZERO-wireclaw**](https://github.com/Nicolboy/VSCODE-PIZERO-wireclaw) | Pi Zero 2W embarqué — caméra OV5647, H.264 en RTP/UDP |

Un opérateur donne ses instructions en français, à l'écrit ou à l'oral ; un
modèle de langage les interprète et produit les commandes MAVLink vers
ArduPilot. La liaison vidéo passe par wifibroadcast longue portée, décodée au
sol sous GStreamer avec détection annotée en temps réel.

---

## Un portage entre deux architectures

| | |
|---|---|
| [**CCS-MSP430-pwl**](https://github.com/Nicolboy/CCS-MSP430-pwl) | alimentation fantôme 48 V sur 1x pile AA, MSP430G2252 |
| [**CCS-MSPM0-pwl**](https://github.com/Nicolboy/CCS-MSPM0-pwl) | le même, porté sur MSPM0C1104 — Cortex-M0+ |

Élévation 1V vers 48 V par DC/DC polyphasé sur un microcontrôleur à 20 broches,
avec **auto-étalonnage sauvegardé en flash** : plutôt que de figer un rapport
cyclique calculé, chaque carte se règle sur ses propres composants sous charge
connue.

Le dépôt MSPM0 garde le journal du portage, périphérique par périphérique, dans
l'ordre où ils ont été franchis.

---

## Outils

| Domaine | |
|---|---|
| Électronique | **KiCad 10**, kicad-cli, API kipy |
| Microcontrôleurs TI | **Code Composer Studio**, C2000Ware, MSPM0 SDK, SysConfig |
| ESP32 | **ESP-IDF**, PlatformIO |
| Linux embarqué | **VS Code**, GStreamer, MAVLink/ArduPilot, YOLOv8 |
| Général | C, Python, C++, git, Claude Code |

---

## Une façon de travailler

Une part du travail est faite avec un agent : lecture de datasheets, extraction
des contraintes, rédaction des spécifications, génération de symboles, pose des
composants. **Le placement et le routage restent manuels** — ils encodent des
choix qu'aucun script ne devine sur une chaîne de mesure de courant.

C'est documenté, avec les pièges rencontrés et leur signature, dans
[methode-kicad-claude.md](https://github.com/Nicolboy/KICAD-C2000-devkit/blob/main/doc/methode-kicad-claude.md).

---

<sub>contact@nicolasboyer.fr</sub>
