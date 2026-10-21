# CarPlay in the Audi Virtual Cockpit (MHI2Q): install guide

A step-by-step guide to putting Apple CarPlay's map and turn-by-turn directions into the Audi
Virtual Cockpit on cars with the **MHI2Q** (MIB2 High, Qualcomm) head unit. That covers many
2017–2020 A4/S4/A5/S5 (B9), Q5 (FY) and Q7 (4M) cars.

The mod itself is **[luka-dev/mib2q-carplay-rgi](https://github.com/luka-dev/mib2q-carplay-rgi)**,
and all the credit for it goes there. This repo only explains how to install it, start to finish,
from someone who has done it on a 2018 S5. It includes the firmware update most European cars
need first, and the places where it is easy to go wrong.

> [!CAUTION]
> This modifies the firmware and configuration of your infotainment unit, at your own risk. It can
> leave the unit unable to boot, and recovery may mean removing the unit and using a serial cable.
> It may also affect your warranty. Nothing here is affiliated with Audi, Volkswagen Group, Apple or
> the authors of the tools linked. This guide redistributes no firmware or binaries: every download
> comes from its original source.

Written 2026-10-21 against mod release `MHI2Q-2026-10-19.1` and M.I.B. `V3.8.0`. **Check
the mod's [releases](https://github.com/luka-dev/mib2q-carplay-rgi/releases) for anything newer**:
it moves quickly, and its release notes come first.

## Contents

- [What you get](#what-you-get)
- [1. Check your car is compatible](#1-check-your-car-is-compatible)
- [2. What you need](#2-what-you-need)
- [3. Button combinations](#3-button-combinations)
- [4. Update the head unit firmware (if needed)](#4-update-the-head-unit-firmware-if-needed)
- [5. Prepare the M.I.B. + CarPlay card](#5-prepare-the-mib--carplay-card)
- [6. Install M.I.B., then the CarPlay mod](#6-install-mib-then-the-carplay-mod)
- [7. Things not to do](#7-things-not-to-do)
- [8. Updating to a newer release](#8-updating-to-a-newer-release)
- [9. Logs, uninstall and recovery](#9-logs-uninstall-and-recovery)
- [10. Limitations](#10-limitations)
- [Credits](#credits)

## What you get

With your iPhone plugged in by cable (see [Limitations](#10-limitations)):

- **CarPlay's own map in the Virtual Cockpit.** The **View** button switches between the classic and
  large layouts, and the steering-wheel roller zooms.
- **Turn-by-turn in the cockpit** from Apple Maps (and other apps that send CarPlay route guidance):
  a maneuver arrow, distance bar, lane arrows, arrival time and the next road name. The same arrows
  appear on the head-up display if you have one.
- **CarPlay calls on the cockpit phone tab**, album art on the media tab, the MMI touchpad moving the
  CarPlay focus, and a short press of the voice button opening Siri.
- **5.1 sound for CarPlay music on cars with an external amplifier (B&O verified).** Cars with the
  head unit's own amplifier stay on stock stereo.

The [project README](https://github.com/luka-dev/mib2q-carplay-rgi#-features) has the full list
and photos.

## 1. Check your car is compatible

All of these must be true:

| Check | How |
| --- | --- |
| The head unit is **MHI2Q**, not plain MHI2 | In the MMI's version information, the media driver version contains **`MMX2QC`** (for example `CLU8_MMX2QC_AU_ER_G22_…`). Plain MHI2 shows `CLU6_MMX2_…` instead, and this mod does not support it |
| You know the firmware **train** and **MU** | Red Engineering Menu → *Version information*, for example `MHI2Q_ER_AUG22_P4170` and software version `1117` |
| Your region is **US, ER, JP or CN** | The second field of the train (`_ER_` = Europe/rest of world) |
| The firmware is a **verified version** | US `P5087`, ER `P5092`, CN `K1004`, JP `P1017`. Older firmware must be updated first ([step 4](#4-update-the-head-unit-firmware-if-needed)) |
| A **fully digital cluster** (Virtual Cockpit) | Analog clusters are not supported |
| **Stock CarPlay already works** on the main screen | Plug your iPhone in. The mod extends Audi's CarPlay; it does not activate it |

The installer double-checks most of this: it reads the train from the unit and refuses to run on
anything other than an Audi MHI2Q, without changing anything. Even so, check first. On a plain
MHI2 unit, see [edzenis/mib2high-carplay-rgi](https://github.com/edzenis/mib2high-carplay-rgi)
instead.

## 2. What you need

- **An SD card**, 32 or 64 GB, that you can wipe. One card is enough for every step.
- **A computer that can write the card as FAT32.** Company-managed laptops often mount SD cards
  read-only. If formatting fails with errors like `Couldn't open device`, use another machine
  (any Linux box with a USB card reader works).
- **A battery charger or maintainer** connected to the car for the whole job. Two hours of ignition
  without the engine will flatten a battery. A small trickle charger may not keep up; see the engine
  note in [step 6](#6-install-mib-then-the-carplay-mod).
- **About 2–3 hours**: 30–60 minutes for the firmware update and 30–45 for the rest, plus
  downloads and card preparation.
- A USB cable for your iPhone.

## 3. Button combinations

The two rocker switches either side of MENU/BACK are toggles: **NAV/MAP ↕ TEL** on the left and
**RADIO ↕ MEDIA** on the right. The key-combination charts online use arrows that show **which way
to push the rocker**, not which half to press. Reading them the wrong way round just opens the phone
or radio screen.

For the B9 A4/A5/S5 and the Q5 (2017–2020) controls:

| To open | Push and hold (a few seconds) | |
| --- | --- | --- |
| **Red Engineering Menu** | left rocker **up** (NAV/MAP) + right rocker **down** (MEDIA) | confirmed |
| **Green Engineering Menu** (GEM) | left rocker **up** (NAV/MAP) + right rocker **up** (RADIO) | confirmed, but only works once enabled ([step 6](#6-install-mib-then-the-carplay-mod)) |
| **Reboot the MMI** | both rockers **up** + press the **rotary knob** | from the chart, not tested |

Other models (Q7 4M, A6/A7 C7) use different combinations. See mibvault's
[Audi key combinations](https://mibvault.com/wiki/general-information/audi-mmi-tastenkombinationen).

## 4. Update the head unit firmware (if needed)

Skip this step if your train is already the verified one for your region (for example
`MHI2Q_ER_AUG22_P5092`, MU 1329).

The mod replaces some of the head unit's own Java classes and an audio driver, and those are only
verified on specific firmware. On older firmware the head unit can get stuck on the Audi logo
([#24](https://github.com/luka-dev/mib2q-carplay-rgi/issues/24),
[#50](https://github.com/luka-dev/mib2q-carplay-rgi/issues/50)). **Only update within the same
train** (`MHI2Q_ER_AUG22_…` to `MHI2Q_ER_AUG22_…`): that is the kind of update the unit accepts normally.

### 4.1 Download

The firmware is on [mibvault.com](https://mibvault.com/files), under **MHI2 - HARMAN → Firmware →
Audi**. The European/RoW (`ER`) file sits directly in the `Audi` folder; the `CN`, `JP` and `US`
subfolders hold the others.

| File | Size | SHA-256 |
| --- | --- | --- |
| `MHI2Q_ER_AUG22_P5092_MU1329 (A4-A5-Q5-Q7).7z` | 2,059,847,610 bytes (1.9 GB) | `e46f3be615dc227f1f127ed26a5aaf0b4d343f37efac072e917a667073de022f` |

For other regions, each file's checksum is in its info (ⓘ) panel on mibvault.

> [!IMPORTANT]
> **Verify the checksum.** The free download is capped at 50 Mbps (about 5–6 minutes), the link
> expires shortly after you click, and the server can **neither resume nor report the file size**. A
> download that slows down can stop early while still looking complete. Ours stopped at 1.53 GB with
> no error the first time.
>
> ```sh
> shasum -a 256 "MHI2Q_ER_AUG22_P5092_MU1329 (A4-A5-Q5-Q7).7z"   # macOS
> sha256sum     "MHI2Q_ER_AUG22_P5092_MU1329 (A4-A5-Q5-Q7).7z"   # Linux
> ```

### 4.2 Write the card

Format the card as **FAT32 on an MBR partition table**, then extract the archive so that
**`metainfo2.txt` is at the top level of the card**. In this archive it already is. Unpacked, it is
about 3.5 GB and 642 files, and no file is over FAT32's 4 GB limit.

**Linux.** Find the card by its USB transport, not by size: a Raspberry Pi's own boot card is the
same size as a 64 GB card.

```sh
lsblk -o NAME,SIZE,TRAN,RM,MODEL            # the card is the TRAN=usb, RM=1 disk, e.g. /dev/sda
DEV=/dev/sdX                                # set this to your card. Everything on it is erased
                                            # (unmount it first if your desktop auto-mounted it)
sudo wipefs -a "$DEV"
printf 'label: dos\ntype=c\n' | sudo sfdisk "$DEV"   # MBR, one FAT32 (LBA) partition
sudo mkfs.vfat -F 32 -n MIB "${DEV}1"
sudo mkdir -p /mnt/mib && sudo mount -o uid=$(id -u) "${DEV}1" /mnt/mib
7z x -o/mnt/mib "MHI2Q_ER_AUG22_P5092_MU1329 (A4-A5-Q5-Q7).7z"
sync && sudo umount /mnt/mib
```

**macOS:**

```sh
diskutil list                                        # find the card, e.g. /dev/disk4
diskutil eraseDisk FAT32 MIB MBRFormat /dev/diskN    # erases that disk
# extract the .7z with 7-Zip (7zz) or The Unarchiver, then copy its contents to /Volumes/MIB
dot_clean -m /Volumes/MIB                            # remove macOS ._ files
diskutil eject /Volumes/MIB
```

**Windows:** format with a tool that does FAT32 on cards over 32 GB (for example Rufus or
guiformat), then extract with 7-Zip straight to the card's root.

If you can, read the files back and compare them with the archive (`7z t`, or CRCs from
`7z l -slt`) before you go to the car.

### 4.3 Flash it in the car

1. Connect the charger. Ignition **on**, engine **off** (don't run the engine during the flash).
   With keyless entry, **keep the key inside the car**: walking away with it switches the ignition
   off and aborts the update. Unplug your phone.
2. Insert the card in **SD slot 1** (in the glovebox on the B9).
3. Red Engineering Menu → **Update** → SD card → **Standard** → scroll to the bottom → **Update**.
   - **B&O or Bose:** owners are advised to leave the **amplifier** out of the update (untick it).
   - Without an external amplifier there is nothing to untick.
4. **Leave it alone.** Reports range from about 15 to 60 minutes, depending on the card's speed,
   with several restarts. Don't remove the card, open the doors, start the engine or turn the
   ignition off. A black screen or a long pause is normal.
5. At the end you may see one component as **NOK**. On our car it was a **"Toolbox"** entry, which
   isn't part of this firmware at all, probably an add-on from a previous owner. If every
   component *in the firmware* is OK, choose **Resume**. Retrying an entry the card has no files for
   can't succeed.
6. **"Start version compare / connect diagnostic tester"** → press **Cancel**. This is the
   dealer's software-registration step (SVM). Cancelling logs an SVM fault and restarts the unit;
   that is expected, and M.I.B. clears it later.
7. Red menu → *Version information* should now show the new train, for example
   `MHI2Q_ER_AUG22_P5092` and MU `1329`. Check that radio, navigation and CarPlay still work.

## 5. Prepare the M.I.B. + CarPlay card

The mod installs through **M.I.B. (More Incredible Bash)**, a toolkit that runs in the head unit's
Green Engineering Menu from the SD card.

1. **Download M.I.B.** from
   [Mr-MIBonk/M.I.B._More-Incredible-Bash](https://github.com/Mr-MIBonk/M.I.B._More-Incredible-Bash/releases).
   We used `V3.8.0`, the newest tag, which has the *Run Custom Script* entry the mod's instructions
   use. It is published as a source archive only: *Source code (zip)*.
2. **Download the mod** from its [latest release](https://github.com/luka-dev/mib2q-carplay-rgi/releases/latest):
   `install_*.zip`, plus `uninstall_*.zip` and `logging_*.zip` to keep for later. Check each against
   the **sha256** shown next to it on the release page.
3. **Wipe the card** and format it FAT32 again, as in [4.2](#42-write-the-card). The firmware files
   are no longer needed on it.
4. **Copy the *contents* of M.I.B.'s folder** (`M.I.B._More-Incredible-Bash-3.8.0/`) to the
   card's root. The card should then hold `metainfo2.txt`, `Launcher/`, `apps/`, `esd/`, `mod/` and
   so on.
5. **Copy the *contents* of the mod's `install_MoreIncredibleBash/`** onto the card, merging into
   M.I.B.'s `mod/` folder:
   - `mod/custom.sh` **replaces** M.I.B.'s placeholder (an empty "put commands here" script). This
     is expected.
   - `mod/command.sh` and `mod/carplay/` are added.
6. Eject the card cleanly (on macOS, run `dot_clean -m` first).

> [!NOTE]
> `uninstall_*.zip` and `logging_*.zip` also contain a `mod/custom.sh`, so **only one of install,
> uninstall or logging can be on the card at a time**. Keep the other two on your computer.

## 6. Install M.I.B., then the CarPlay mod

Same setup as the flash: charger on, ignition on, key inside, **phone unplugged**, card in SD slot 1.
Leave the card in for the whole session: the **m.i.b** entry only appears in the Green menu while
the card is in.

> [!TIP]
> **From here the engine may run.** The firmware flash is over, and what follows only copies files and
> edits settings. If the car warns about low battery, start the engine **between steps, never while a
> script is running** (cranking can restart the MMI), and give the MMI a minute to settle. Do this
> outdoors.

1. **Install M.I.B.:** Red menu → **Update** → SD card → **"FREE for all - M.I.B. Launcher V1.2"** →
   start. The unit restarts three times (5–10 minutes). The summary should show the Launcher as
   done. If the diagnostic-tester / version-compare screen appears again, press **Cancel**. This
   install also switches on the Green Engineering Menu.
2. **Open the Green menu:** NAV/MAP up + RADIO up. After a cold boot it can take **up to 5 minutes**
   before it opens. You should see entries such as **production** and **mib**.
3. **Make a backup:** mib → Backup → **"System backup - MANDATORY before patching and other
   operations"** → **Backup ~5 minutes**. It saves the unit's EEPROM, feature activations (FEC),
   storage and versions to the card's `backup/` folder. **Copy that folder somewhere safe**
   afterwards. M.I.B. invites you to upload it to mibsolution.one; that's optional, and it is your car's
   own identity and activation data.
4. **Clear the SVM fault:** run M.I.B.'s **SVM fix**.
5. **Install the mod:** with the phone still unplugged, go to mib → **Advanced Settings → Run Custom
   Script** (older M.I.B. releases call it *Run individual script*; the package works with both).
   Take a photo of the output. You want to see:
   - `Firmware: MHI2Q_ER_AUG22_P5092 (platform MHI2Q, region ER, brand AU)`, or your own train
   - an `Amplifier:` line (*internal … stays stock stereo* is normal without B&O/Bose)
   - at the end, after about 20 seconds: **`DONE (install). Reboot the HU to load.`**

   If a line says **`FAILED`**, stop: the installer changes nothing when its checks fail. If it
   **stops with a warning** (firmware not verified, or another CarPlay mod found), read it. Running
   the script again in the same power cycle installs anyway, so only do that if you're sure.
6. **Reboot the MMI** (both rockers up + knob, or switch the car off and on). The mod's Java part only
   loads on a full restart.
7. **Plug your iPhone in.** CarPlay starts as usual, and the cockpit shows the CarPlay map. Start an
   Apple Maps route to check the arrows and lanes. After a cold boot the cockpit may say
   *"Initialising navigation"* for a few seconds; that's normal.
8. Take the card out. The mod doesn't need it.

## 7. Things not to do

- **Don't run M.I.B.'s "CarPlay Maps VC".** M.I.B. V3.8.0 bundles a *different* CarPlay-cluster mod
  for VW/Skoda/SEAT units. It touches the same parts of the system, and two CarPlay mods on one unit
  is a documented way to break this one.
- **Don't install other CarPlay cluster mods** (MIBSI, mibr, the CarPlay AltScreen toolbox).
  Their uninstallers leave files behind that stop this mod working
  ([#39](https://github.com/luka-dev/mib2q-carplay-rgi/issues/39),
  [#53](https://github.com/luka-dev/mib2q-carplay-rgi/issues/53)). The installer now stops with a
  warning if it finds one. If one was ever installed, the clean fix is to reflash the stock firmware first.
- **Don't enable NavActiveIgnore / "dual navigation"** in M.I.B. The mod deliberately removes it: it
  hides CarPlay's state from the head unit.
- **Leave ifs-root / FEC patching alone** unless you want it for other reasons, such as map updates. It is
  not needed for this, and it carries its own risk.

## 8. Updating to a newer release

No uninstall is needed. Install the new version over the old one:

1. Download the new `install_*.zip` and check its sha256.
2. On your M.I.B. card, **replace `mod/carplay/` completely** and overwrite `mod/custom.sh` and
   `mod/command.sh` with the new ones. The release notes say to replace the whole `mod/` folder,
   but on a card that also carries M.I.B., that folder holds M.I.B.'s own files, so replace only the
   mod's parts.
3. Phone unplugged → Green menu → mib → Advanced Settings → **Run Custom Script** → wait for `DONE` → reboot.

## 9. Logs, uninstall and recovery

- **Logs for a bug report:** swap the contents of `logging_MoreIncredibleBash/` onto the card in
  place of the install files and run it the same way. Run it once, reconnect the phone and reproduce
  the problem, then run it again. The second numbered folder under `carplay_logs/` on the card holds
  the detailed session. Attach it to an issue on the
  [mod's tracker](https://github.com/luka-dev/mib2q-carplay-rgi/issues).
- **Uninstall:** do the same with `uninstall_MoreIncredibleBash/`. It restores every file the
  installer replaced from the `.carplay-stock` backups it made, removes what it added, and is safe
  to run twice. Then reboot.
- **Stuck on the Audi logo after an install:** the mod's repo has a rescue card that removes the
  Java part without opening the unit (`rescue_MoreIncredibleBash/`). The worst case reported needed
  the head unit removed and a serial console ([#24](https://github.com/luka-dev/mib2q-carplay-rgi/issues/24)).

## 10. Limitations

- **Wired only.** These units only support CarPlay over USB; wireless CarPlay arrived with the
  later MMI generations. The mod's own wireless research is
  [stalled](https://github.com/luka-dev/mib2q-carplay-rgi/issues/41) on the unit's Wi-Fi speed
  (about 9 Mbps stable against ~19 Mbps needed). Plug-in wireless adapters are hit and miss
  ([#43](https://github.com/luka-dev/mib2q-carplay-rgi/issues/43)). A Carlinkit 5.0 connects, but
  audio plays from the phone, the music tab is missing, and the small cockpit map doesn't resize.
  A no-name adapter didn't work at all.
- **5.1 sound needs an external amplifier** (B&O verified, Bose untested). On the head unit's own
  amplifier CarPlay audio stays stereo; real 5.1 there is still
  [research](https://github.com/luka-dev/mib2q-carplay-rgi/issues/62).
- **Turn-by-turn depends on the app.** Apple Maps sends full route guidance. Waze doesn't, so it
  draws its own turn card into the cockpit map instead.

## Credits

- **[luka-dev/mib2q-carplay-rgi](https://github.com/luka-dev/mib2q-carplay-rgi)**: the mod, and the
  extensive documentation this guide leans on (GPL-3.0).
- **[M.I.B. – More Incredible Bash](https://github.com/Mr-MIBonk/M.I.B._More-Incredible-Bash)**: the
  head-unit toolkit used to install it.
- **[MIB Vault](https://mibvault.com)**: the firmware archive and key-combination charts.
