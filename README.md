# Firmware OTA

Ce dossier contient les firmwares compiles destines a etre deposes sur le serveur OTA.

## Contenu actuel

- `firmware-1.0.1.bin` : binaire ESP32-S3 (FIRMWARE_VERSION = "1.0.1"), ~1.7 MB
- `manifest.json` : description de la version disponible (lue par l'app Flutter)

## SHA256

```
0d65009e3d56df77628069c1ad692f628727b9caae83c082a01ad3514409ae83  firmware-1.0.1.bin
```

Verifier localement :
```bash
shasum -a 256 firmware-1.0.1.bin
```

## Procedure de test

### 1. Mettre en place le serveur (GitHub Pages)

Creer un repo public `horalux-ota` (par exemple https://github.com/fpchalopin/horalux-ota) avec :
- `firmware-1.0.1.bin`
- `manifest.json`

Activer GitHub Pages sur la branche `main` (Settings > Pages > Source = main, dossier = root).
Apres ~1 min, les fichiers sont accessibles sous `https://<user>.github.io/horalux-ota/`.

### 2. Cote ESP32

S'assurer que l'ESP32 est flashe en version 1.0.0 :
```bash
pio run -t upload
```
Le code source est en 1.0.0 (defini dans `src/main.cpp` : `#define FIRMWARE_VERSION "1.0.0"`).

### 3. Cote app Flutter

L'URL du manifest est definie dans `app/lib/screens/update_screen.dart` :
```dart
const String _manifestUrl =
    'https://fpchalopin.github.io/horalux-ota/manifest.json';
```
Modifier si necessaire pour pointer sur ton repo.

### 4. Test

1. Lancer l'app Flutter, se connecter a HoraLux
2. Menu > Mise a jour
3. La "Version installee" doit afficher `1.0.0`
4. Cliquer "Verifier les mises a jour"
5. La carte doit afficher "Nouvelle version 1.0.1 disponible" + le changelog

Le bouton "Installer" est encore un stub (etape 3 du plan OTA non encore implementee).

## Procedure pour generer une nouvelle version

1. Bumper `FIRMWARE_VERSION` dans `src/main.cpp` (ex: "1.0.2")
2. Compiler : `pio run`
3. Copier : `cp .pio/build/esp32s3/firmware.bin bin/firmware-1.0.2.bin`
4. Calculer le hash : `shasum -a 256 bin/firmware-1.0.2.bin`
5. Mettre a jour `manifest.json` avec version, url, sha256, size, changelog
6. Revert `FIRMWARE_VERSION` dans `src/main.cpp` a la version installee actuellement (sinon a la prochaine compilation/flash, le code source bumpe l'ESP32 et il ne verra plus de mise a jour)
7. Push sur le repo OTA
