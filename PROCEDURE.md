# OnePlus 7T HD1903 — Procédure complète Root avec Magisk 30.7

> Appareil : OnePlus 7T HD1903 (hotdogb)
>
> OS : OxygenOS 11 / Android 11
>
> Magisk : 30.7
>
> Méthode : obtention du boot stock → patch Magisk → test temporaire avec `fastboot boot` → sauvegarde `dd` du boot stock (une fois root temporaire actif) → installation persistante avec Magisk Direct Install.

---

## ⚠️ Avertissement

**Je ne suis pas responsable si vous ne suivez pas les étapes dans l'ordre.**

Sauter une étape ou en inverser l'ordre mène généralement au **Qualcomm CrashDump**.

---

# 0. Prérequis

Sur le PC :

- ADB installé
- Fastboot installé
- câble USB fonctionnel
- téléphone avec bootloader déverrouillé
- Magisk 30.7 APK
- le ZIP/firmware OxygenOS 11 correspondant exactement à la build installée (pour en extraire `boot.img`)

Vérifier ADB :

```bash
adb version
```

Vérifier Fastboot :

```bash
fastboot --version
```

---

# 1. Vérifier que le téléphone est correctement détecté

Depuis le PC :

```bash
adb devices
```

Résultat attendu :

```text
XXXXXXXX    device
```

Si le téléphone demande :

```text
Allow USB debugging?
```

accepter sur le téléphone.

---

# 2. Vérifier le modèle et la version Android

```bash
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.display.id
```

Référence de notre appareil :

```text
OnePlus7T
Android 11
OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
```

> ⚠️ Le `boot.img` utilisé pour le patch doit provenir **exactement** de cette build.

---

# 3. Vérifier le slot actif

Avant de toucher au boot, toujours vérifier le slot actif :

```bash
adb shell getprop ro.boot.slot_suffix
```

Exemple :

```text
_b
```

> IMPORTANT :
>
> Ne pas supposer que le slot est `_b`.
> Toujours vérifier avant d'extraire ou de restaurer une partition.

Dans notre procédure, le slot actif était :

```text
_b
```

---

# 4. Installer Magisk 30.7

Depuis le PC :

```bash
adb push Magisk-v30.7.apk /sdcard/Download/
```

Vérifier :

```bash
adb shell ls -lh /sdcard/Download/Magisk-v30.7.apk
```

Sur le téléphone :

```text
Gestionnaire de fichiers
→ Download
→ Magisk-v30.7.apk
→ Installer
```

Ouvrir ensuite Magisk.

Vérifier depuis le PC :

```bash
adb shell pm list packages | grep -i magisk
```

Résultat attendu :

```text
package:com.topjohnwu.magisk
```

---

# 5. État du root à ce stade

À ce stade, Magisk est installé mais le téléphone **n'est pas encore rooté**.

Tester :

```bash
adb shell su -c id
```

Il est **normal** que cette commande ne retourne pas encore :

```text
uid=0(root)
```

Le root sera obtenu à l'étape 12 (`fastboot boot`).

> C'est précisément pour cette raison que le `dd` ne peut pas être fait maintenant.

---

# 6. Obtenir le boot.img stock (SANS root)

Puisque `su` n'est pas disponible, le boot stock doit venir du firmware, pas du téléphone.

Depuis le ZIP OxygenOS correspondant à la build installée :

```bash
python3 payload_dumper.py --partitions boot payload.bin
```

ou extraire `boot.img` du package de la même façon selon l'outil utilisé.

Vérifier :

```bash
ls -lh boot.img
sha256sum boot.img
```

Renommer pour clarté :

```bash
mv boot.img boot_oos11-stock.img
```

> Ce fichier est le boot stock **du firmware**.
> Le boot stock **de l'appareil** sera sauvegardé plus tard avec `dd` (étape 14).

---

# 7. Envoyer le boot stock sur le téléphone

```bash
adb push boot_oos11-stock.img /sdcard/Download/boot_oos11-stock.img
```

Vérifier :

```bash
adb shell ls -lh /sdcard/Download/boot_oos11-stock.img
```

---

# 8. Patch du boot avec Magisk

Dans Magisk :

```text
Install
→ Select and Patch a File
```

Sélectionner :

```text
/sdcard/Download/boot_oos11-stock.img
```

Magisk patch le boot.

À la fin, Magisk indique le chemin du fichier créé.

Généralement :

```text
/sdcard/Download/magisk_patched-xxxxx.img
```

---

# 9. Vérifier le fichier patché

Depuis le PC :

```bash
adb shell ls -lt /sdcard/Download/magisk_patched*.img
```

Noter le nom exact.

Exemple :

```text
magisk_patched-30700_a8HWn.img
```

---

# 10. Récupérer le boot patché

```bash
adb pull /sdcard/Download/magisk_patched-xxxxx.img .
```

Vérifier :

```bash
ls -lh magisk_patched-xxxxx.img
sha256sum magisk_patched-xxxxx.img
```

---

# 11. NE PAS flasher directement

⚠️ Étape critique.

Ne pas faire immédiatement :

```bash
fastboot flash boot magisk_patched-xxxxx.img
```

Notre première tentative de flash direct avait conduit au mode :

```text
Qualcomm CrashDump
```

La stratégie retenue est donc d'abord de **tester le boot temporairement**.

## ⚠️ Solution alternative risquée — Ajustement manuel de la taille du boot

Dans certains cas, une image `boot.img` extraite directement de l'appareil peut présenter une différence de taille ou d'alignement par rapport à l'image attendue par l'outil de flash. Une solution expérimentale consiste à ajuster manuellement la taille du fichier en supprimant quelques octets à sa fin afin de respecter l'alignement requis.

> ⚠️ **Cette méthode est risquée et ne doit pas être considérée comme une procédure normale.**
>
> La suppression d'octets modifie le fichier binaire. Même si les octets supprimés semblent correspondre uniquement à une zone de padding en fin d'image, il n'est pas garanti qu'ils soient inutiles pour tous les formats ou toutes les versions de boot.
>
> Une image modifiée de cette manière peut provoquer un échec de démarrage, un bootloop ou, dans le pire cas, un **Qualcomm CrashDump**.
>
> **Toujours conserver une copie intacte du boot stock avant toute modification et privilégier le patch Magisk + `fastboot boot` comme méthode de test.**

---

# 12. Démarrer TEMPORAIREMENT avec le boot patché

## 12.1 Redémarrer en Bootloader / Fastboot

```bash
adb reboot bootloader
```

Vérifier :

```bash
fastboot devices
```

Résultat attendu :

```text
XXXXXXXX    fastboot
```

## 12.2 Vérifier le slot Fastboot

```bash
fastboot getvar current-slot 2>&1
```

Vérifier que le slot correspond au boot que l'on a préparé.

Dans notre procédure :

```text
b
```

## 12.3 Boot temporaire

⚠️ Ne pas utiliser `flash`.

Utiliser :

```bash
fastboot boot magisk_patched-xxxxx.img
```

Le téléphone doit redémarrer.

Attendre le démarrage complet d'Android.

---

# 13. Vérifier le root temporaire

Depuis le PC :

```bash
adb devices
```

Puis :

```bash
adb shell su -c id
```

Résultat attendu :

```text
uid=0(root)
```

Vérifier également :

```bash
adb shell su -c 'magisk -c'
```

Exemple :

```text
30.7:MAGISK:R (30700)
```

À ce stade :

```text
Boot patché
    ↓
Démarrage OK
    ↓
Magisk OK
    ↓
Root TEMPORAIRE OK
```

---

# 14. Extraire le boot STOCK du téléphone avec `dd` (maintenant possible)

C'est **ici**, et seulement ici, que le `dd` devient réalisable : le root temporaire est actif.

Cette sauvegarde correspond au boot **réellement présent sur l'appareil**, ce qui est plus fiable qu'un boot téléchargé d'une autre version d'OxygenOS.

Entrer dans le shell :

```bash
adb shell
```

Passer root :

```bash
su
```

Vérifier :

```bash
id
```

Résultat attendu :

```text
uid=0(root)
```

## 14.1 Vérifier la partition boot

Pour le slot `_b` utilisé dans notre procédure :

```bash
ls -l /dev/block/bootdevice/by-name/boot_b
```

On doit obtenir un lien vers une partition bloc.

## 14.2 Faire une copie du boot stock

```bash
dd if=/dev/block/bootdevice/by-name/boot_b of=/sdcard/boot_b-stock.img
```

Attendre la fin de `dd`.

Vérifier la taille :

```bash
ls -lh /sdcard/boot_b-stock.img
```

Puis quitter le shell root :

```bash
exit
exit
```

> ⚠️ Faire ce `dd` **avant** le Direct Install de l'étape 15, sinon on sauvegarde un boot déjà patché et non le boot stock.

## 14.3 Récupérer le boot stock sur le PC

```bash
adb pull /sdcard/boot_b-stock.img .
```

Vérifier :

```bash
ls -lh boot_b-stock.img
sha256sum boot_b-stock.img
```

> `boot_b-stock.img` est notre sauvegarde STOCK de référence.
>
> Ne pas la modifier.
>
> La conserver comme point de restauration.

---

# 15. Installation PERSISTANTE avec Magisk

Une fois que le boot patché a démarré correctement, que `su` fonctionne et que la sauvegarde `dd` est faite :

Ouvrir Magisk :

```text
Install
→ Direct Install (Recommended)
```

Laisser Magisk terminer l'installation.

Cette étape installe Magisk de manière persistante sur le boot du slot actif.

---

# 16. Redémarrer Android

Depuis le PC :

```bash
adb reboot
```

Attendre le démarrage complet.

---

# 17. Vérification finale du root

```bash
adb shell su -c id
```

Résultat attendu :

```text
uid=0(root)
```

---

# 18. Vérifier la version Magisk

```bash
adb shell su -c 'magisk -c'
```

Référence :

```text
30.7:MAGISK:R (30700)
```

---

# 19. Vérifier le slot

```bash
adb shell getprop ro.boot.slot_suffix
```

Dans notre configuration :

```text
_b
```

---

# 20. Vérifier SELinux

```bash
adb shell getenforce
```

Résultat possible :

```text
Enforcing
```

ou, selon la configuration :

```text
Permissive
```

> Les deux ne signifient pas automatiquement que le root fonctionne ou ne fonctionne pas.
> Le test principal est `su -c id`.

---

# 21. Vérifications finales

```bash
adb shell su -c id
adb shell su -c 'magisk -c'
adb shell getprop ro.boot.slot_suffix
adb shell getprop ro.build.display.id
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
```

Référence :

```text
uid=0(root)
30.7:MAGISK:R (30700)
_b
OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
OnePlus7T
11
```

---

# 22. Fichiers importants à conserver sur le PC

Conserver au minimum :

```text
boot_oos11-stock.img       (boot stock issu du firmware, utilisé pour le patch)
boot_b-stock.img           (boot stock extrait de l'appareil avec dd — RÉFÉRENCE)
magisk_patched-xxxxx.img   (boot patché testé et fonctionnel)
```

Le fichier le plus important est :

```text
boot_b-stock.img
```

Il correspond exactement au boot original de notre installation.

---

# 23. Résumé de la procédure (ordre corrigé)

```text
ONEPLUS 7T HD1903
       │
       ▼
OxygenOS 11
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
adb push → téléphone
       │
       ▼
Magisk → Select and Patch a File
       │
       ▼
magisk_patched-xxxxx.img
       │
       ▼
adb pull → PC
       │
       ▼
adb reboot bootloader
       │
       ▼
fastboot boot   (PAS flash)
       │
       ▼
DÉMARRAGE TEMPORAIRE
       │
       ▼
su -c id  →  ROOT TEMPORAIRE OK
       │
       ▼
dd → boot_b-stock.img   (sauvegarde du vrai boot stock)
       │
       ▼
adb pull → PC
       │
       ▼
Magisk → Direct Install (Recommended)
       │
       ▼
adb reboot
       │
       ▼
ROOT PERSISTANT
       │
       ▼
NetHunter
```

---

# 24. Règle de sécurité principale

```text
BOOT STOCK (firmware)
    ↓
PATCH MAGISK
    ↓
FASTBOOT BOOT
    ↓
TEST / ROOT TEMPORAIRE
    ↓
dd → SAUVEGARDE BOOT STOCK RÉEL
    ↓
MAGISK DIRECT INSTALL
```

Éviter de passer directement de :

```text
boot.img
    ↓
fastboot flash boot
```

sans avoir vérifié la compatibilité et sans disposer d'une sauvegarde.

---

# 25. Méthode de restauration

Si un boot patché pose problème et que le téléphone peut encore accéder au bootloader, utiliser uniquement une image STOCK correspondant exactement à la partition/au slot concerné.

Ne jamais utiliser au hasard :

```text
boot.img
```

provenant d'une autre version d'OxygenOS.

Toujours vérifier :

```bash
fastboot getvar current-slot 2>&1
```

avant toute opération sur le boot.

---

# 26. Points clés à ne pas oublier

1. Vérifier le modèle/build Android avant de commencer.
2. Vérifier le slot actif avant toute opération sur le boot.
3. **Le `dd` nécessite le root → il vient APRÈS `fastboot boot`, pas avant.**
4. Le boot utilisé pour le patch initial provient du **firmware**, pas du téléphone.
5. Conserver `boot_b-stock.img` comme sauvegarde immuable.
6. Calculer les SHA-256 des images importantes.
7. Tester avec `fastboot boot` avant toute installation persistante.
8. Ne pas présenter `Permissive` comme une condition obligatoire du root.
9. Séparer clairement « boot temporaire » et « Direct Install ».
10. Documenter le problème CrashDump rencontré avec le flash direct.
11. Faire le `dd` avant le Direct Install, sinon on sauvegarde un boot déjà patché.
12. Vérifier le slot avant une éventuelle restauration.

---

> **Méthode de référence :**
> On récupère le boot stock du firmware, on le patch avec Magisk, on le teste temporairement avec `fastboot boot`. Une fois le root temporaire obtenu, on sauvegarde le vrai boot stock de l'appareil avec `dd`. Puis seulement, on fait `Direct Install (Recommended)` dans Magisk pour rendre le root persistant.
