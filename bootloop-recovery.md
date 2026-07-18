# Bootloop Recovery

> [!WARNING]
> Please proceed at your own risk. There are chances that performing below
> instructions could lead to unrecoverable phone.
> The author is not responsible for any damage caused to the phone with
> below instructions.

## Was passiert ist

Ein Bootloop trat auf, nachdem `boot.img` mit einer **aktuellen Magisk-Version
(30.x)** gepatcht wurde. [`rootphone.md`](./rootphone.md) dokumentiert
explizit nur **Magisk v27.0** als getestet funktionierend – eine neuere
Version führte in diesem Fall zu einem Bootloop.

Zusätzlich fehlten beim ursprünglichen Patch-Versuch zwei Schritte aus
`rootphone.md`, die vermutlich zur Instabilität beigetragen haben:
- `fastboot flash --disable-verity --disable-verification vbmeta vbmeta.img`
- Flashen der gepatchten `boot.img` auf **beide** Slots (`boot_a` **und**
  `boot_b`), nicht nur den aktiven.

## Voraussetzung für die Wiederherstellung

Ein Backup des Original-`boot.img` **vor** dem Patchen ist zwingend
Voraussetzung – siehe [`backupbootimg.md`](./backupbootimg.md). Ohne dieses
Backup ist der Weg unten nicht möglich (dann bleibt nur ein Stock-ROM-Flash,
siehe [`flashstockrom.md`](./flashstockrom.md)).

## Wiederherstellung

1. Ins Fastboot booten (siehe [`unlockbootloader.md`](./unlockbootloader.md)
   für die Tastenkombination).
2. Aktiven Slot ermitteln:
   ```
   fastboot getvar current-slot
   ```
3. Original-`boot.img` auf den aktiven Slot zurückspielen:
   ```
   fastboot flash boot_<slot> boot_<slot>_stock_backup.img
   fastboot reboot
   ```
   Das Gerät sollte jetzt wieder normal (ungerootet) booten.

## Korrektes Re-Root

1. **Magisk v27.0** installieren (nicht die neueste Version –
   [Download](https://github.com/topjohnwu/Magisk/releases/tag/v27.0)).
2. In der Magisk-App: Install → "Select and Patch a File" → das gesicherte
   Original-`boot.img` auswählen.
3. Gepatchte Datei zurückholen, dann **exakt** wie in
   [`rootphone.md`](./rootphone.md) beschrieben flashen – insbesondere:
   ```
   fastboot flash --disable-verity --disable-verification vbmeta vbmeta.img
   fastboot flash boot_a magisk_patched_boot.img
   fastboot flash boot_b magisk_patched_boot.img
   fastboot reboot
   ```
4. Root mit `su -c id` verifizieren (`uid=0(root)` erwartet).

## Praktischer Hinweis: USB-Verbindungsprobleme während des Flashens

Bei größeren Transfers über einen USB-Hub/eine Dockingstation können
sporadische `Protocol error`/`Broken pipe`-Fehler bei `fastboot flash`
auftreten (auch bei kleinen Dateien wie `vbmeta.img`). Falls das passiert:
direkt an einen USB-Port anschließen (nicht über Hub/Dock) und den
Flash-Befehl erneut ausführen – idempotent, schadet nicht bei Wiederholung.
