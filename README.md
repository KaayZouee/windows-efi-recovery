# Tutorial on how to get your Windows EFI back!!

*A guide by KaayZouee.*

Have you ever accidentally formatted your **`nvme0n1p1`** while installing a Linux distro?
That sounds really horrible!!
But don’t worry, I gotchu.

⚠️ **Disclaimer:** This guide only restores the **EFI boot partition**. If you accidentally formatted or deleted your main Windows partition (for example, `nvme0n1p3` in my case), your data may be unrecoverable with this method.
Your hardware and OS may behave differently, so results are not guaranteed.

---

## Step 1: Download the Windows ISO

You’ll need a Windows installation ISO to repair EFI.

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

We’ll use **WoeUSB** to flash the ISO. You can use other tools (Rufus on Windows, balenaEtcher, etc.), but here’s the NixOS way:

```bash
nix-shell -p woeusb ntfs3g --run \
"sudo woeusb --device /home/kay/Downloads/Win11_24H2_EnglishInternational_x64.iso /dev/sda"
```

⚠️ **Warning:** This will erase everything on `/dev/sda` (your USB drive). Backup first.

When it finishes, unmount the USB safely. Now reboot your computer.

---

## Step 3: Boot from the USB

* As your PC restarts, enter the **boot menu** (usually `Esc`, `F2`, `F12`, or `Del`, depending on your manufacturer).

  * Example: On my Asus Vivobook S14, I spam **Esc** and hold **F2**.
* Choose the USB device to boot.

You should now see the Windows installer screen.

---

## Step 4: Open Command Prompt

* On the Windows setup screen, don’t click *Install now*.
* Instead, press **Shift + F10** → this opens a **Command Prompt**.

---

## Step 5: Recreate the EFI Partition

If your EFI partition (`nvme0n1p1`) is completely wiped, you need to recreate it.

1. Launch `diskpart`:

   ```cmd
   diskpart
   ```
2. List your disks:

   ```cmd
   list disk
   ```

   Identify your main disk (usually `Disk 0`).
3. Select your disk:

   ```cmd
   select disk 0
   ```
4. List partitions:

   ```cmd
   list partition
   ```
5. Create a new EFI partition (100MB is enough):

   ```cmd
   create partition efi size=100
   format quick fs=fat32
   assign letter=S
   exit
   ```

Now you have a clean EFI partition mounted as `S:`.

---

## Step 6: Repair Boot Files

Run the following commands:

```cmd
bcdboot C:\Windows /s S: /f UEFI
```

Explanation:

* `C:\Windows` → your Windows system partition (usually C:).
* `/s S:` → tells Windows to copy boot files to the EFI partition (S:).
* `/f UEFI` → ensures the system boots in UEFI mode.

If successful, it should say:
**Boot files successfully created.**

---

## Step 7: Reboot

Close Command Prompt, remove the USB, and reboot.
If everything worked, your Windows bootloader should be back!

---

## Optional: Dual Boot with Linux

If you’re dual booting:

* Boot back into Linux.
* Run:

  ```bash
  sudo update-grub
  ```

  or regenerate your bootloader config (depending on your distro, e.g., `grub-mkconfig` on Arch, `nixos-rebuild` on NixOS).

This ensures GRUB detects Windows again.

---

## Troubleshooting

* If `bcdboot` fails, check that:

  * Your Windows partition is still intact.
  * The EFI partition is formatted as FAT32.
* If Windows doesn’t boot, go back to USB and run **Startup Repair** from the Windows installer.

---

✅ And that’s it. Your Windows EFI is restored without reinstalling the whole OS!

---
