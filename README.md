# Formatif F1 — Introduction au Raspberry Pi et capteurs Adafruit

**Cours** : 243-413-SH — Introduction aux objets connectes
**Semaine** : 1
**Type** : Formative (non notee)
**Retries** : Illimites - poussez autant de fois que necessaire!

---

## Progressive Milestones

Ce formatif utilise des **jalons progressifs** avec retroaction detaillee:

| Jalon | Points | Verification |
|-------|--------|-------------|
| **Milestone 1** | 25 pts | Script existe, syntaxe valide, tests locaux executes |
| **Milestone 2** | 35 pts | I2C initialise, capteur AHT20 cree, lecture temperature/humidite |
| **Milestone 3** | 40 pts | Fonction main(), gestion d'erreurs, qualite du code |

**Chaque test echoue vous dit**:
- Ce qui etait attendu
- Ce qui a ete trouve
- Une suggestion pour corriger

---

## Objectif

Ce formatif vise a verifier que vous etes capable de :
1. Creer une cle SSH sur le Raspberry Pi et l'ajouter a votre compte GitHub
2. Installer UV et gerer les dependances Python
3. Detecter un capteur I2C avec `i2cdetect`
4. Lire un capteur AHT20 (temperature, humidite)
5. Controler un NeoSlider (potentiometre + LEDs) - optionnel

---

## Workflow de soumission

```
+-----------------------------------------------------------------+
|                    WORKLOAD FORMATIF F1                          |
+-----------------------------------------------------------------+
|                                                                  |
|  1. Sur le Raspberry Pi (via SSH avec mot de passe)              |
|     +-- Creer une cle SSH                                        |
|     +-- Afficher la cle publique                                 |
|                                                                  |
|  2. Sur GitHub (via navigateur)                                  |
|     +-- Ajouter la cle SSH a votre compte                        |
|     +-- Tester la connexion (ssh -T git@github.com)              |
|                                                                  |
|  3. Sur le Raspberry Pi                                          |
|     +-- Installer UV                                             |
|     +-- Cloner votre depot GitHub (avec URL SSH)                 |
|     +-- Creer test_aht20.py                                     |
|     +-- Executer: python3 run_tests.py                           |
|     +-- Corriger les erreurs                                     |
|     +-- Pousser: git add, commit, push                           |
|                                                                  |
|  4. GitHub Actions valide automatiquement                        |
|     +-- Verifie les marqueurs de tests                           |
|     +-- Confirme que vous avez tout complete                     |
|                                                                  |
+-----------------------------------------------------------------+
```

---

## Instructions detaillees

### Etape 1 : Creer une cle SSH sur le Raspberry Pi

Connectez-vous d'abord au Raspberry Pi avec votre mot de passe :

```bash
ssh utilisateur@HOSTNAME.local
```

Puis, generez une cle SSH **directement sur le Raspberry Pi** :

```bash
# Generer la cle avec un commentaire identifiant
ssh-keygen -t ed25519 -C "iot-cegep@etu.cegep.qc.ca" -f ~/.ssh/id_ed25519_iot
```

- Appuyez **Entree** pour accepter l'emplacement par defaut
- Appuyez **Entree** deux fois pour laisser la passphrase vide

#### Afficher la cle publique

```bash
cat ~/.ssh/id_ed25519_iot.pub
```

Copiez **toute** la ligne affichee (commence par `ssh-ed25519 ...`)

---

### Etape 2 : Ajouter la cle SSH a votre compte GitHub

1. Allez sur https://github.com et connectez-vous
2. Cliquez sur votre photo -> **Settings**
3. Menu gauche -> **SSH and GPG keys**
4. Cliquez sur **New SSH key**
5. Remplissez :
   - **Title** : `Raspberry Pi IoT - Cours 243-413-SH`
   - **Key** : Collez la cle publique copiee
   - **Key type** : Authentication Key
6. Cliquez sur **Add SSH key**

#### Configurer SSH pour GitHub

Toujours sur le Raspberry Pi :

```bash
# Ajouter la cle a l'agent SSH
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_iot

# Creer un config pour utiliser cette cle avec GitHub
cat > ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_iot
    IdentitiesOnly yes
EOF

# Securiser le fichier config
chmod 600 ~/.ssh/config
```

#### Tester la connexion avec GitHub

```bash
ssh -T git@github.com
```

**Resultat attendu** (si succes) :
```
Hi votrenom! You've successfully authenticated, but GitHub does not provide shell access.
```

> Bravo ! Votre cle SSH est configuree et vous pouvez maintenant cloner et pousser directement depuis le Raspberry Pi !

---

### Etape 3 : Installer UV et cloner le depot

Une fois la cle SSH configuree :

```bash
# Installer UV
curl -LsSf https://astral.sh/uv/install.sh | sh

# Recharger le shell
source ~/.bashrc

# Configurer Git (IMPORTANT!)
git config --global user.name "Prenom Nom"
git config --global user.email "votre.email@cegepsherbrooke.qc.ca"
git config --global init.defaultbranch main
```

```bash
# Cloner votre depot GitHub Classroom avec l'URL SSH
git clone git@github.com:tge-sherbrooke/semaine-1-f1-votre-username.git
cd semaine-1-f1-votre-username
```

> **Note** : Utilisez l'URL **SSH** affichee sur GitHub (commence par `git@github.com:`)

---

### Etape 4 : Activer I2C et verifier les capteurs

```bash
# Activer I2C
sudo raspi-config nonint do_i2c 0

# Installer les outils I2C
sudo apt update && sudo apt install -y i2c-tools

# Scanner le bus I2C
sudo i2cdetect -y 1
```

Vous devriez voir :
- `38` pour le AHT20
- `30` pour le NeoSlider

**IMPORTANT** : Les capteurs fonctionnent UNIQUEMENT en 3.3V !

---

### Etape 5 : Creer et tester le AHT20

Creez le fichier `test_aht20.py` :

```python
# /// script
# requires-python = ">=3.9"
# dependencies = ["adafruit-circuitpython-ahtx0", "adafruit-blinka"]
# ///
"""Test du capteur AHT20 via STEMMA QT/I2C."""

import board
import adafruit_ahtx0

i2c = board.I2C()
sensor = adafruit_ahtx0.AHTx0(i2c)

print(f"Temperature: {sensor.temperature:.1f} C")
print(f"Humidite: {sensor.relative_humidity:.1f} %")
```

Testez-le :

```bash
uv run test_aht20.py
```

---

### Etape 6 : Creer et tester le NeoSlider (optionnel)

Creez le fichier `test_neoslider.py` :

```python
# /// script
# requires-python = ">=3.9"
# dependencies = ["adafruit-circuitpython-seesaw", "adafruit-blinka"]
# ///
"""Test du NeoSlider - Animation arc-en-ciel sur les LEDs."""

import board
import time
from rainbowio import colorwheel
from adafruit_seesaw.seesaw import Seesaw
from adafruit_seesaw import neopixel

# Configuration NeoSlider
i2c = board.I2C()
neoslider = Seesaw(i2c, 0x30)
pixels = neopixel.NeoPixel(neoslider, 14, 4, pixel_order=neopixel.GRB)

# Position dans la roue des couleurs
color_pos = 0

while True:
    pixels.fill(colorwheel(color_pos))
    color_pos = (color_pos + 1) % 256
    time.sleep(0.02)
```

Testez-le :

```bash
uv run test_neoslider.py
```

---

### Etape 7 : Executer les tests locaux

**Ceci est l'etape obligatoire avant de pousser!**

```bash
python3 run_tests.py
```

Le script `run_tests.py` va :
1. Verifier que votre cle SSH existe
2. Verifier que `test_aht20.py` est correct
3. Verifier que `test_neoslider.py` est correct (optionnel)
4. Scanner le bus I2C pour detecter les capteurs
5. Creer des fichiers marqueurs dans `.test_markers/`

Si tous les tests passent, vous verrez :
```
TOUS LES TESTS SONT PASSES!
```

---

### Etape 8 : Pousser votre travail

Une fois les tests passes :

```bash
git add .
git commit -m "feat: tests AHT20 et NeoSlider completes"
git push
```

GitHub Actions validera automatiquement que vous avez execute les tests.

---

## Cablage STEMMA QT

| Fil | Raspberry Pi |
|-----|--------------|
| Rouge (VIN) | 3.3V |
| Noir (GND) | GND |
| Bleu (SDA) | GPIO 2 |
| Jaune (SCL) | GPIO 3 |

**VIN doit etre connecte a 3.3V, PAS 5V !**

---

## Comprendre la validation

### Pourquoi executer `run_tests.py` AVANT de pousser ?

Le formatif F1 utilise une validation en deux temps :

| Etape | Ou | Ce qui est valide |
|-------|----|-------------------|
| **run_tests.py** | Sur Raspberry Pi | - Cle SSH creee sur le Pi<br>- Connexion GitHub fonctionnelle<br>- Scripts crees<br>- Capteurs detectes |
| **GitHub Actions** | Automatique apres push | - Les marqueurs existent<br>- Syntaxe Python valide |

Cette approche garantit que vous avez **reellement** travaille sur le materiel tout en beneficiant de l'automatisation GitHub.

### Que se passe-t-il si je pousse sans executer les tests ?

GitHub Actions affichera une erreur :
```
ERREUR: Les tests locaux n'ont pas ete executes!
```

Vous devrez alors executer `python3 run_tests.py` sur le Raspberry Pi et repousser.

---

## Livrables

Dans ce depot, vous devez avoir :

- [ ] `test_aht20.py` -- Script de lecture du capteur AHT20
- [ ] `test_neoslider.py` -- Script de test du NeoSlider (optionnel)
- [ ] `.test_markers/` -- Dossier cree par `run_tests.py` (ne pas editer manuellement!)

---

## Resume des commandes

```bash
# ===== SUR RASPBERRY PI (connexion initiale) =====
ssh utilisateur@HOSTNAME.local

# ===== CREER LA CLE SSH =====
ssh-keygen -t ed25519 -C "iot-cegep@etu.cegep.qc.ca" -f ~/.ssh/id_ed25519_iot

# ===== AFFICHER LA CLE (a copier pour GitHub) =====
cat ~/.ssh/id_ed25519_iot.pub

# ===== AJOUTER LA CLE A GITHUB =====
# Allez sur https://github.com -> Settings -> SSH and GPG keys -> New SSH key

# ===== CONFIGURER SSH SUR LE PI =====
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_iot
cat > ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_iot
    IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config

# ===== TESTER LA CONNEXION GITHUB =====
ssh -T git@github.com

# ===== INSTALLER UV =====
curl -LsSf https://astral.sh/uv/install.sh | sh && source ~/.bashrc

# ===== CONFIGURER GIT =====
git config --global user.name "Prenom Nom"
git config --global user.email "votre.email@etu.cegep.qc.ca"

# ===== CLONER LE DEPOT (AVEC URL SSH) =====
git clone git@github.com:tge-sherbrooke/semaine-1-f1-votre-username.git
cd semaine-1-f1-votre-username

# ===== ACTIVER I2C =====
sudo raspi-config nonint do_i2c 0
sudo apt install -y i2c-tools

# ===== SCANNER I2C =====
sudo i2cdetect -y 1

# ===== TESTER LES CAPTEURS =====
uv run test_aht20.py
uv run test_neoslider.py

# ===== EXECUTER LES TESTS =====
python3 run_tests.py

# ===== POUSSER =====
git add .
git commit -m "feat: tests completes"
git push
```

---

## Ressources

- [Guide de l'etudiant](../deliverables/activites/semaine-1/labo/guide-etudiant.md)
- [Guide de depannage](../deliverables/activites/semaine-1/labo/guide-depannage.md)

---

Bonne chance !
