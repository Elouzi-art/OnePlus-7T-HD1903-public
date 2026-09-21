# OnePlus 7T HD1903 — Root Magisk 30.7 + NetHunter

**Documentation de référence et fichiers pour rooter un OnePlus 7T HD1903 sous OxygenOS 11, puis installer Kali NetHunter.**

| | |
|---|---|
| **Appareil** | OnePlus 7T HD1903 (`hotdogb`) |
| **ROM** | OxygenOS 11 / Android 11 |
| **Build de référence** | `OnePlus7TOxygen_14.E.35_GLO_0350_2206171459` |
| **Magisk** | 30.7 (30700) |
| **Méthode** | Patch Magisk → `fastboot boot` (test) → `dd` (sauvegarde) → Direct Install |
| **Statut** | Testé et fonctionnel |

---

## Pourquoi ce dépôt existe

La variante **HD1903** (Inde / global) du OnePlus 7T est mal couverte par la documentation existante :

- Les `boot.img` stock pour cette variante et cette build sont **quasiment introuvables** en ligne. La plupart des liens des forums sont morts, hébergés sur des services expirés, ou correspondent à d'autres variantes (HD1901, HD1905, HD1907) ou à d'autres builds d'OxygenOS.
- Flasher un `boot.img` provenant d'une autre variante ou d'une autre build mène très rapidement au mode **Qualcomm CrashDump**.
- Les guides disponibles présentent souvent les étapes dans un ordre logiquement impossible (extraction `dd` du boot **avant** d'avoir le root — voir la note ci-dessous).

Ce dépôt regroupe donc :

1. Une **procédure vérifiée de bout en bout**, avec l'ordre des étapes corrigé.
2. Les **fichiers nécessaires**, pour ne pas dépendre de liens morts.
3. Les **pièges rencontrés** et la manière de s'en sortir.

---

## Le point qui bloque tout le monde

De nombreux tutoriels demandent d'extraire le boot stock avec :

```bash
dd if=/dev/block/bootdevice/by-name/boot_b of=/sdcard/boot_b-stock.img
```

Cette commande nécessite `su`, **donc le root** — que l'on n'a pas encore à ce stade. C'est un problème de l'œuf et de la poule.

L'ordre correct, appliqué dans ce dépôt :

```text
1. Obtenir un boot.img stock SANS root (extrait du firmware OOS)
2. Le patcher avec Magisk
3. fastboot boot            → ROOT TEMPORAIRE
4. dd                       → sauvegarde du vrai boot stock de l'appareil
5. Magisk → Direct Install  → ROOT PERSISTANT
```

Le `dd` reste indispensable, mais comme **sauvegarde de référence**, pas comme point de départ. Et il doit être fait **avant** le Direct Install, sinon on sauvegarde un boot déjà patché.

---

## Contenu du dépôt

```text
.
├── README.md
├── docs/
│   └── PROCEDURE.md              # Procédure complète, étape par étape
├── files/
│   ├── Magisk-v30.7.apk
│   ├── boot_oos11-stock.img      # boot stock issu du firmware OOS 11
│   └── nethunter/
│       ├── NetHunterStore.apk
│       ├── com.offsec.nethunter_XXXXXXXXXX.apk
│       ├── com.offsec.nhterm_XXXXXXXXXX.apk
│       └── NetHunterKeX.apk
└── checksums/
    └── SHA256SUMS
```

> Vérifiez systématiquement les empreintes avant de flasher quoi que ce soit :
>
> ```bash
> sha256sum -c checksums/SHA256SUMS
> ```

---

## Prérequis

- PC avec `adb` et `fastboot` installés
- Câble USB fonctionnel (éviter les rallonges et les hubs)
- Bootloader déverrouillé
- Débogage USB activé
- **Le firmware OxygenOS correspondant exactement à votre build installée**

Vérification préalable, à faire avant toute chose :

```bash
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.display.id
adb shell getprop ro.boot.slot_suffix
```

Si votre `ro.build.display.id` diffère de la build de référence ci-dessus, **n'utilisez pas le `boot_oos11-stock.img` de ce dépôt** : extrayez celui de votre propre firmware.

---

## Procédure

La procédure complète et commentée se trouve dans **[`docs/PROCEDURE.md`](docs/PROCEDURE.md)**.

Résumé du flux :

```text
ONEPLUS 7T HD1903
       │
       ▼
Vérifier modèle / build / slot actif
       │
       ▼
Installer Magisk 30.7 (pas encore root)
       │
       ▼
boot.img extrait du FIRMWARE OOS 11
       │
       ▼
Magisk → Select and Patch a File
       │
       ▼
magisk_patched-xxxxx.img → adb pull
       │
       ▼
fastboot boot   (PAS flash)
       │
       ▼
su -c id  →  ROOT TEMPORAIRE OK
       │
       ▼
dd → boot_b-stock.img  (sauvegarde)
       │
       ▼
Magisk → Direct Install (Recommended)
       │
       ▼
ROOT PERSISTANT
       │
       ▼
NetHunter
```

### Règle de sécurité

Ne jamais faire directement :

```bash
fastboot flash boot magisk_patched-xxxxx.img
```

Toujours tester d'abord :

```bash
fastboot boot magisk_patched-xxxxx.img
```

`boot` est temporaire : si l'image est mauvaise, un simple redémarrage vous ramène à l'état précédent. `flash` est définitif — c'est ce qui nous a envoyés en Qualcomm CrashDump lors du premier essai.

---

## Installation de NetHunter

Une fois le root persistant obtenu :

```bash
adb push files/nethunter/NetHunterStore.apk /sdcard/Download/
adb push files/nethunter/com.offsec.nethunter_XXXXXXXXXX.apk /sdcard/Download/
adb push files/nethunter/com.offsec.nhterm_XXXXXXXXXX.apk /sdcard/Download/
adb push files/nethunter/NetHunterKeX.apk /sdcard/Download/
```

Puis installer chaque APK depuis le gestionnaire de fichiers.

### Pourquoi les APK séparés plutôt que l'image complète

Passer par le téléchargement de l'image NetHunter complète depuis kali.org représente **environ 2 Go** et **une quinzaine de minutes** au minimum, pour un résultat identique si vous ne voulez que l'environnement de base.

Installer directement les APK — **NetHunter Store**, **NetHunter** et **NetHunter Terminal** — fournis ici évite ce téléchargement. Le chroot Kali peut ensuite être ajouté à la demande, uniquement si vous en avez réellement besoin.

### Après l'installation

Pensez à mettre à jour le **kernel compatible OSS v2** depuis l'application NetHunter.

---

## Dépannage

### Qualcomm CrashDump

Ce n'est pas une brique définitive. La procédure de récupération est le **MSM Download Tool (Windows)**, largement documenté et facile à trouver : il restaure l'appareil en usine via le mode EDL.

Points à retenir :

- Nécessite Windows et les pilotes Qualcomm.
- Efface intégralement l'appareil.
- Après un MSM, l'appareil repart sur une ancienne version d'OxygenOS : il faut remettre à jour vers OOS 11 avant de relancer cette procédure.
- Le bootloader redevient verrouillé — il faut refaire `fastboot oem unlock`.

### Le téléphone n'apparaît pas

```bash
adb devices      # doit afficher "device", pas "unauthorized"
fastboot devices # en mode bootloader
```

Accepter la fenêtre « Allow USB debugging » sur le téléphone. Sur Linux, vérifier les règles udev.

### `su -c id` ne retourne rien

Normal **avant** l'étape `fastboot boot`. Après, c'est le signe que le boot patché n'a pas démarré correctement : redémarrez et repartez d'un `boot.img` correspondant exactement à votre build.

### SELinux affiche `Enforcing`

Ce n'est pas un problème. `Enforcing` comme `Permissive` sont compatibles avec le root. Le seul test qui compte est :

```bash
adb shell su -c id
```

---

## Notes complémentaires

- **Magisk fourni.** Une version récente de Magisk (30.7, icône orange) est incluse dans `files/`. Si vous avez besoin d'une autre version ou d'un autre build, ouvrez une issue : elle pourra être ajoutée au dépôt.
- **Batterie et optimisations diverses.** Les recommandations d'usage courantes (gestion de la batterie, modules Magisk, réglages de performance) sont abondamment couvertes ailleurs, notamment en vidéo sur YouTube. Ce dépôt ne les duplique pas et se concentre sur ce qui manque réellement : le boot stock HD1903 et une procédure fiable.
- **Périmètre.** Ce dépôt documente le root et l'installation de NetHunter. Il ne couvre pas les ROM custom, TWRP ni les recoveries alternatifs.

---

## Contribuer

Les contributions sont bienvenues, en particulier :

- Confirmations de fonctionnement sur d'autres builds d'OxygenOS 11
- `boot.img` stock d'autres variantes (HD1901, HD1905, HD1907) avec leur empreinte SHA-256
- Corrections et précisions sur la procédure

Merci d'indiquer systématiquement dans vos issues :

```text
Modèle        : (ro.product.model)
Build         : (ro.build.display.id)
Version       : (ro.build.version.release)
Slot actif    : (ro.boot.slot_suffix)
Version Magisk:
Étape bloquée :
```

---

## Avertissement

Rooter un appareil annule la garantie, peut empêcher certaines applications de fonctionner (banque, paiement sans contact, DRM) et comporte un risque de mise hors service temporaire.

Toutes les opérations décrites ici sont effectuées **sous votre entière responsabilité**. Les auteurs de ce dépôt ne sauraient être tenus responsables d'un appareil endommagé ou de données perdues.

**Sauvegardez vos données avant de commencer.**

---

## Licence

Documentation publiée sous [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

Magisk est un projet de John Wu, distribué sous licence GPL-3.0.
Kali NetHunter est un projet d'OffSec.
Les fichiers de firmware restent la propriété de OnePlus / OPPO et sont redistribués ici à des fins de restauration et d'interopérabilité uniquement.
