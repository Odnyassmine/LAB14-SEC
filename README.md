# LAB14-SEC
# Contournement de Détection Root Android avec Frida, Objection et Medusa

## 1. Présentation du laboratoire

Ce laboratoire a pour objectif de comprendre les mécanismes de détection de root Android et les techniques de neutralisation dans un environnement de test contrôlé.

Le laboratoire couvre :

- l’installation de Frida et Objection,
- le déploiement de frida-server,
- les bases de l’instrumentation dynamique Android,
- le bypass de détection root Java,
- le bypass de détection root native (C/C++),
- l’utilisation simplifiée via Objection,
- l’introduction à Medusa,
- les cas où Magisk devient nécessaire.

> ⚠️ Ce laboratoire doit être réalisé uniquement sur un appareil de laboratoire ou un émulateur autorisé.

---

# 2. Objectifs pédagogiques

À la fin du laboratoire, vous serez capable de :

- Installer Frida et Objection
- Configurer ADB et le débogage USB
- Déployer frida-server sur Android
- Comprendre les hooks Frida
- Bypasser des détections root Java
- Bypasser certaines vérifications natives
- Utiliser Objection pour automatiser les bypass
- Comprendre les limites des bypass runtime

---

# 3. Prérequis

## Logiciels nécessaires

- Python 3
- pip
- ADB (Android Platform Tools)
- Frida
- Objection
- Téléphone Android ou émulateur
- 7-Zip (Windows)

---

## Connaissances recommandées

- Android Debug Bridge (ADB)
- Java Android
- Linux basique
- Android rooting
- HTTPS et sécurité mobile

---

# 4. Architecture du laboratoire

```text
Android App
      │
      ▼
+------------------+
|      Frida       |
|   frida-server   |
+------------------+
      │
      ▼
+------------------+
|    Objection     |
|     Medusa       |
+------------------+
      │
      ▼
Hooks Runtime Android
```

---

# 5. Étape 1 — Préparer l’environnement

# 5.1 Installer Python et Frida

## Vérifier Python

```bash
python --version
pip --version
```

---

## Installer Frida

```bash
pip install --upgrade frida frida-tools
```

---

## Vérifications

```bash
frida --version
python -c "import frida; print(frida.__version__)"
```

---

## Astuce Windows

Si :

```text
frida is not recognized
```

Ajouter au PATH :

```text
%USERPROFILE%\AppData\Roaming\Python\Python311\Scripts
```

---

## Installation sûre

```bash
python -m pip install --upgrade frida frida-tools
```

---

# 5.2 Installer ADB

## Télécharger Android Platform Tools

```text
https://developer.android.com/tools/releases/platform-tools
```

---

## Vérifier ADB

```bash
adb version
adb devices
```

---

## Résultat attendu

```text
device
```

et non :

```text
unauthorized
```

---

# Activer le débogage USB

## Android

```text
Paramètres
→ À propos du téléphone
→ Taper 7× sur "Numéro de build"
```

Puis :

```text
Paramètres
→ Système
→ Options développeur
→ Débogage USB
```

---

## Accepter l’empreinte USB

Valider :

```text
Autoriser le débogage USB
```

---

## Drivers Windows

Installer :

- Samsung USB Driver
- Google USB Driver
- OnePlus / Xiaomi drivers

---

## Erreurs fréquentes

- USB non reconnu
- Driver absent
- Mode USB incorrect

---

# 6. Étape 2 — Démarrer frida-server

## Objectif

Permettre l’instrumentation Android depuis le PC.

---

# 6.1 Identifier l’architecture CPU

```bash
adb shell getprop ro.product.cpu.abi
```

---

## Exemples

```text
arm64-v8a
armeabi-v7a
x86_64
```

---

# 6.2 Télécharger frida-server

## Source officielle

```text
https://github.com/frida/frida/releases
```

---

## Important

La version doit correspondre exactement :

```text
frida --version
=
frida-server version
```

---

## Décompression

### Windows

Utiliser :

```text
7-Zip
```

---

### Linux / macOS

```bash
tar xf frida-server-*.xz
```

---

# 6.3 Envoyer et lancer frida-server

## Push

```bash
adb push frida-server /data/local/tmp/
```

---

## Permissions

```bash
adb shell chmod 755 /data/local/tmp/frida-server
```

---

## Lancement

```bash
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"
```

---

## Arrière-plan

```bash
adb shell "nohup /data/local/tmp/frida-server -l 0.0.0.0 >/dev/null 2>&1 &"
```

---

## Forward ports

```bash
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

---

## Vérification

```bash
frida-ps -Uai
```

---

## Résultat attendu

Liste des applications Android visibles.

---

## Erreurs fréquentes

- Mauvaise architecture CPU
- Version mismatch
- frida-server non exécutable

---

# 7. Étape 3 — Comprendre Frida rapidement

## Concepts clés

| Option | Fonction |
|---|---|
| -U | Appareil USB |
| -f | Spawn application |
| -n | Attach application |
| Java.perform | Hook Java |

---

## Script test hello.js

```javascript
Java.perform(function () {
  console.log("[+] Script injecté: Java.perform OK");
});
```

---

## Injection

```bash
frida -U -f <package> -l hello.js --no-pause
```

---

## Résultat attendu

```text
[+] Script injecté: Java.perform OK
```

---

## Attach sur app ouverte

```bash
frida -U -n "NomDuProcessus" -l hello.js
```

---

## Pourquoi ?

Certaines applications crashent lors du spawn.

---

# 8. Étape 4 — Bypass Root Java avec Frida

## Détections root courantes

Applications Android recherchent souvent :

- Build.TAGS = test-keys
- fichiers su
- busybox
- Runtime.exec("su")
- bibliothèques RootBeer

---

## Script bypass_root_basic.js

```javascript
// Hooks simples pour bypass root
Java.perform(function () {
  console.log("[+] Bypass Java installé");
});
```

> Remplacer par le script complet fourni dans le TP.

---

## Injection

```bash
frida -U -f <package> -l bypass_root_basic.js --no-pause
```

---

## Résultat attendu

- plus d’alerte root,
- logs Frida visibles,
- application fonctionnelle.

---

## Hooks réalisés

### Build.TAGS

```text
release-keys
```

---

### File.exists()

Blocage des chemins :

```text
/system/bin/su
/system/xbin/su
busybox
```

---

### Runtime.exec()

Blocage :

```text
su
which su
busybox
```

---

### RootBeer

```text
isRooted() → false
```

---

## À observer

- logs [+]
- bypass actif
- absence de détection root

---

## Erreurs fréquentes

- mauvais package
- app crash
- hooks trop tardifs

---

# 9. Étape 4.1 — Bypass Root Natif (C/C++)

## Pourquoi ?

Certaines applications utilisent du code natif pour détecter le root.

---

## Fonctions natives ciblées

```text
open
openat
access
stat
lstat
```

---

## Script bypass_native.js

```javascript
// Hooks libc natifs
console.log("[+] Native hooks installés");
```

> Remplacer par le script complet fourni dans le TP.

---

## Injection combinée

```bash
frida -U -f <package> -l bypass_root_basic.js -l bypass_native.js --no-pause
```

---

## Résultat attendu

Blocage des accès :

```text
/system/bin/su
/proc/mounts
busybox
```

---

## Découverte dynamique

Utiliser :

```bash
frida-trace -U -i open -i access -i stat -i openat -i fopen -i readlink <package>
```

---

## À observer

- chemins recherchés
- appels libc
- logs de blocage

---

# 10. Étape 5 — Utiliser Objection

## Objectif

Automatiser les bypass Frida.

---

# 10.1 Installer Objection

## Méthode recommandée

```bash
pip install --user pipx
pipx ensurepath
pipx install objection
```

---

## Alternative

```bash
pip install --upgrade objection
```

---

## Vérification

```bash
objection --version
```

---

# 10.2 Utilisation

## Spawn + bypass automatique

```bash
objection -g <package> explore --startup-command "android root disable"
```

---

## Attach sur app ouverte

```bash
objection -g <package> explore
```

Puis :

```bash
android root disable
```

---

## Ce que fait Objection

- patch Build.TAGS
- neutralise File.exists()
- bloque Runtime.exec()
- patch RootBeer

---

## Limites

Certaines apps natives nécessitent Frida natif complémentaire.

---

## À observer

- hooks actifs
- logs Objection
- absence de root détecté

---

## Erreurs fréquentes

- frida-server absent
- package incorrect
- protections natives agressives

---

# 11. Étape 6 — Utiliser Medusa

## Objectif

Utiliser une autre surcouche Frida.

---

## Installation

```bash
git clone <URL_Medusa>
cd Medusa
pip install -r requirements.txt
```

---

## Aide

```bash
python medusa.py --help
```

---

## Exemples

```bash
python medusa.py --usb --spawn <package> --module root-bypass
```

ou :

```bash
medusa --usb --attach "NomDuProcessus" --module root-bypass
```

---

## À observer

- logs hooks
- bypass root
- stabilité application

---

## Limites

En cas d’échec :

```text
Revenir à Frida pur
```

---

# 12. Étape 7 — Quand utiliser Magisk

## Pourquoi ?

Certaines applications utilisent :

- Play Integrity
- SafetyNet
- propriétés système profondes

---

## Cas où Frida ne suffit plus

Utiliser :

- Magisk
- Zygisk
- DenyList
- modules complémentaires

---

## Étapes résumées

### Activer Zygisk

Dans Magisk.

---

### Configurer DenyList

Inclure :

- application cible
- Play Services
- Play Store

---

### Modules possibles

- Play Integrity Fix
- Shamiko
- MagiskHide Props Config

---

## Limites importantes

```text
Strong Integrity
```

est généralement non contournable sans vulnérabilité système.

---

# 13. Livrables

## Captures recommandées

### Versions

```bash
frida --version
objection --version
adb devices
```

---

### Validation Frida

```bash
frida-ps -Uai
```

---

### Commandes utilisées

Exemple :

```bash
frida -U -f <package> -l bypass_root_basic.js --no-pause
```

---

### Logs bypass

Captures :

- hooks actifs
- bypass Runtime.exec
- bypass File.exists

---

# 14. Bonnes pratiques

✅ Utiliser un appareil de laboratoire  
✅ Documenter versions et commandes  
✅ Utiliser les mêmes versions Frida  
✅ Tester d’abord en Java avant natif  
✅ Limiter les hooks au strict nécessaire

---

# 15. Dépannage rapide

| Problème | Cause probable |
|---|---|
| device unauthorized | empreinte USB non acceptée |
| Frida version mismatch | versions incompatibles |
| app crash au spawn | protections anti-tampering |
| frida-ps vide | frida-server absent |
| bypass inefficace | détection native |

---

# 16. Conclusion

Ce laboratoire introduit les bases du contournement de détection root Android avec Frida, Objection et Medusa.

Les compétences acquises incluent :

- instrumentation dynamique Android,
- hooks Java et natifs,
- neutralisation de vérifications root,
- utilisation d’Objection,
- compréhension des limites runtime.

L’objectif principal reste l’analyse défensive et l’étude des mécanismes de sécurité mobile dans un environnement de test contrôlé.
