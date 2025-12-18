# A quick and short guide on how to get your Windows EFI back!!

*by KaayZouee, for Nix (and maybe, Linux) fans.*
*Yes, I used GPT to rewrite my guide, FIGHT ME.*

Have you ever accidentally formatted your **`nvme0n1p1`** while installing a Linux distro?
That sounds really horrible!!
But don’t worry, I gotchu.

⚠️ **Disclaimer:** This guide only restores the **EFI boot partition**. If you accidentally formatted or deleted your main Windows partition (for example, `nvme0n1p3` in my case), your data may be unrecoverable with this method.
Your hardware and OS may behave differently, so results are not guaranteed.
> Please make sure you have read [this guide](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-the-boot-menu-on-a-dual-boot-pc?view=windows-11) and also [this](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdboot-command-line-options-techref-di?view=windows-11).
---

## Step 1: Download the Windows ISO
You’ll need a Windows installation ISO to repair EFI.
> Since windows already blocked me from downloading their ISO, I used this website. It seems legit, I guess... If you can, just grab the iso directly from the Windows website for extra secure.

* Go to [msdl.gravesoft.dev](https://msdl.gravesoft.dev/) to get the official ISO download links.
* If you’re on an Intel/AMD laptop/desktop, select the standard Windows 11 build (ignore ARM unless you specifically use an ARM-based device like Surface Pro X).
* Pick your product language (e.g., English International).
* Click the blue download link (something like **Windows 11 24H2 English International**) and wait for the download to complete.

Example:

```
/home/kay/Downloads/Win11_24H2_EnglishInternational_x64.iso
```

---

## Step 2: Create a Bootable USB

We’ll use **WoeUSB** to flash the ISO. You can use other tools ([Rufus](https://github.com/pbatard/rufus/tree/master) on Windows, [balenaEtcher](https://github.com/balena-io/etcher), etc.), but here’s the NixOS way:

```bash
nix-shell -p woeusb ntfs3g --run \
"sudo woeusb --device /home/kay/Downloads/Win11_24H2_EnglishInternational_x64.iso /dev/sda"
```

Replace `kay` with your username on Linux.

⚠️ **Warning:** This will erase everything on `/dev/sda` (your USB drive). Backup first.

When it finishes, unmount the USB safely. Now reboot your computer.

---

## Step 3: Boot from the USB

* As your PC restarts, enter the **boot menu** (usually `Esc`, `F2`, `F12`, or `Del`, depending on your manufacturer).

  * Example: On my Asus Vivobook S14, I spam **Esc** and hold **F2**.
* Choose the USB device to boot.

You should now see the Windows installer screen.

---

## Step 4: Enter Windows Recovery Environment (WinRE)

Instead of clicking *Install now*, click **Repair your PC** (bottom-left).

This opens **Windows Recovery Environment (WinRE)**.
Here you can run:

* **Startup Repair** (recommended if you want the simplest fix)
* **System Restore** (if you have a restore point)
* **Command Prompt** (manual repair steps below)

> Note: Using WinRE is safer because it won’t touch your files unless you explicitly reinstall Windows.

---

## Step 5: Prepare the EFI Partition

Open **Command Prompt** from:
*Repair → Troubleshoot → Command Prompt*

Run the following:

```cmd
diskpart
list disk
sel disk 0          # If your Windows is on Disk 0 — pick the correct one!
list vol
```

Look for the **EFI System Partition** (usually 100–300MB, FAT32).
Suppose it’s **Volume 3**, then:

```cmd
sel vol 3
assign letter=Z
exit
```

Now the EFI partition is mounted as `Z:`.

---

## Step 6: Identify Your Windows Partition

Check which drive letter contains Windows:

```cmd
dir C:\
```

If you see folders like `Windows`, `Program Files`, `Users`, then `C:` is correct.
If not, try `dir D:\`, `dir E:\`, etc. until you find your Windows install.

---

## Step 7: Repair Boot Configuration

Run the following commands in order:

```cmd
bootrec /fixmbr
bootrec /scanos
bootrec /rebuildbcd
```
* `bootrec /fixmbr` → rewrites the Master Boot Record (safe for UEFI systems).
* `bootrec /scanos` → scans for Windows installations.
* `bootrec /rebuildbcd` → rebuilds the Boot Configuration Data (BCD).

  * If it finds Windows (like `C:\Windows`), it will ask:
    *“Do you want to add this installation to the boot list (Y/N/A)?”*
  * Press `Y` to include it.

Finally, repair the EFI boot files with:

```cmd
bcdboot C:\Windows /s Z: /f UEFI
```
* `C:\Windows` → tells **bcdboot** where your Windows system files are.
* `/s Z:` → specifies the EFI System Partition (ESP) mounted as `Z:`.
* `/f UEFI` → forces creation of UEFI-style boot entries.

⚠️ **Important:**
Make sure `Z:` is indeed the EFI partition (\~100–300MB, FAT32, usually hidden).
Pointing to the wrong partition could overwrite another EFI loader (e.g., Linux GRUB).

---

## Step 8: Reboot

Close Command Prompt, remove the USB, and reboot.
If all went well, Windows should boot normally again. 🎉

---

## Step 9 (Optional): Dual Boot with Linu
> On my laptop, it already run without step 9, but if you still can't open windows after step 8, try this.
If you’re dual booting:

* Boot into Linux.
* Update your bootloader so it detects Windows again.

**NixOS:**
```
sudo nixos-rebuild switch
```
## Troubleshooting

* If `bootrec /rebuildbcd` doesn’t find your Windows installation, make sure your partition is healthy and not corrupted.
* If Windows still won’t boot, try **Startup Repair** from WinRE.
* If the EFI partition was completely destroyed, you may need to **recreate it** manually (see advanced section in Microsoft docs).

---

✅ Done. Your Windows EFI bootloader should now be restored!

---
