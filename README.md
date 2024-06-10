# Οδηγός εγκατάστασης Arch Linux με κρυπτογράφηση δίσκου

Αυτός ο οδηγός περιέχει τις εντολές που χρησιμοποιήθηκαν στο βίντεο μου στο youtube για την εγκατάσταση του Arch Linux με κρυπτογράφηση δίσκου. Ακολουθήστε τις εντολές βήμα προς βήμα για να ολοκληρώσετε την εγκατάσταση.

Με αυτόν τον οδηγό, θα έχετε ένα πλήρως λειτουργικό και κρυπτογραφημένο σύστημα Arch Linux. Μπορείτε να αντιγράψετε τις εντολές και να τις εκτελέσετε για να ολοκληρώσετε την εγκατάσταση.
Καλή επιτυχία!

## Βήμα 1: Προετοιμασία Δίσκου

### 1. Διαγραφή του υπάρχοντος διαμερίσματος
```
sgdisk -Z /dev/vda
```
Αυτή η εντολή διαγράφει όλα τα υπάρχοντα διαμερίσματα στον δίσκο /dev/vda.



### 2. Δημιουργία νέων διαμερισμάτων
```
sgdisk -n1:0:+512M -t1:ef00 -c1:EFI -N2 -t2:8304 -c2:root /dev/vda
```
Δημιουργία δύο διαμερισμάτων: ένα EFI (512MB) και ένα root με το υπόλοιπο διαθέσιμο χώρο.



### 3. Ενημέρωση του πυρήνα για τις αλλαγές των διαμερισμάτων
```
partprobe -s /dev/vda
```
Ενημερώνει το σύστημα για τα νέα διαμερίσματα.



### 4. Εμφάνιση των διαμερισμάτων
```
lsblk /dev/vda
```
Εμφανίζει τα διαμερίσματα και τις συσκευές δίσκων.



## Βήμα 2: Κρυπτογράφηση Δίσκου

### 1. Διαμόρφωση LUKS
```
cryptsetup luksFormat --type luks2 /dev/vda2
```
Κρυπτογράφηση του διαμερίσματος /dev/vda2 με LUKS.



### 2. Άνοιγμα του κρυπτογραφημένου διαμερίσματος
```
cryptsetup luksOpen /dev/vda2 root
```
Άνοιγμα του κρυπτογραφημένου διαμερίσματος και αντιστοίχιση στο όνομα root.



## Βήμα 3: Δημιουργία Filesystems

### 1. Διαμόρφωση του διαμερίσματος EFI
```
mkfs.vfat -F32 -n EFI /dev/vda1
```
Δημιουργία ενός FAT32 filesystem στο διαμέρισμα EFI.



### 2. Διαμόρφωση του διαμερίσματος root
```
mkfs.btrfs -f -L root /dev/mapper/root
```
Δημιουργία ενός Btrfs filesystem στο κρυπτογραφημένο διαμέρισμα root.



## Βήμα 4: Προσάρτηση Filesystems

### 1. Προσάρτηση του root filesystem
```
mount /dev/mapper/root /mnt
```
Προσάρτηση του root filesystem στο /mnt.



### 2. Δημιουργία και προσάρτηση του διαμερίσματος EFI
```
mkdir /mnt/efi
mount /dev/vda1 /mnt/efi
```
Δημιουργία καταλόγου για το EFI και προσάρτηση του διαμερίσματος EFI σε αυτόν.




## Βήμα 5: Δημιουργία Subvolumes

### 1. Δημιουργία subvolumes
```
btrfs subvolume create /mnt/home
btrfs subvolume create /mnt/srv
btrfs subvolume create /mnt/var
btrfs subvolume create /mnt/var/log
btrfs subvolume create /mnt/var/cache
btrfs subvolume create /mnt/var/tmp
```
Δημιουργία των απαραίτητων subvolumes για το Btrfs.



## Βήμα 6: Ενημέρωση Καθρεπτών Pacman

### 1. Αντιγραφή και Ταξινόμηση Καθρεπτών
```
reflector --country GR --age 24 --protocol http,https --sort rate --save /etc/pacman.d/mirrorlist
```
Ανανεώνει την λίστα των καθρεπτών του pacman για την Ελλάδα.



## Βήμα 7: Εγκατάσταση Βασικών Πακέτων

### 1. Εγκατάσταση Πακέτων
```
pacstrap -K /mnt base base-devel git sof-firmware linux linux-firmware intel-ucode nano cryptsetup btrfs-progs dosfstools util-linux unzip zip sbctl networkmanager sudo openssh xdg-user-dirs
```
Εγκατάσταση των βασικών πακέτων στο νέο σύστημα.

## Βήμα 8: Διαμόρφωση Συστήματος

### 1. Δημιουργία fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```
Δημιουργία του αρχείου fstab για το νέο σύστημα.



### 2. Επεξεργασία του locale.gen
```
nano /mnt/etc/locale.gen
```
Ενεργοποιήστε τις απαιτούμενες γλώσσες και εκτελέστε:

```
arch-chroot /mnt locale-gen
```


### 3. Επεξεργασία του hostname και δημιουργία χρήστη
```
arch-chroot /mnt useradd -G wheel -m to_onoma_sou
arch-chroot /mnt passwd to_onoma_sou
nano /mnt/etc/sudoers
```
Προσθήκη του χρήστη σας στην ομάδα wheel και διαμόρφωση των sudoers.



## Βήμα 9: Διαμόρφωση του πυρήνα

### 1. Προσθήκη εντολών στο cmdline
```
echo "quiet rw" >/mnt/etc/kernel/cmdline
```


### 2. Επεξεργασία του mkinitcpio.conf
```
nano /mnt/etc/mkinitcpio.conf
```
Προσθέστε την γραμμή:
```
HOOKS=(base systemd autodetect modconf kms keyboard keymap sd-vconsole sd-encrypt block filesystems fsck)
```

### 3. Δημιουργία αρχείου preset
```
nano /mnt/etc/mkinitcpio.d/linux.preset
```
Προσθέστε την γραμμή:
```
ALL_microcode=(/boot/*-ucode.img)
```



### 4. Δημιουργία του initramfs
```
arch-chroot /mnt mkinitcpio -P
```


## Βήμα 10: Ενεργοποίηση Υπηρεσιών

### 1. Ενεργοποίηση βασικών υπηρεσιών
```
systemctl --root /mnt enable systemd-resolved systemd-timesyncd NetworkManager sshd
systemctl --root /mnt mask systemd-networkd
```



## Βήμα 11: Εγκατάσταση bootloader

### 1. Εγκατάσταση systemd-boot
```
arch-chroot /mnt bootctl install --esp-path=/efi
```


## Βήμα 12: Επανεκκίνηση στο νέο σύστημα
### 1. Επανεκκίνηση
```
sync
systemctl reboot --firmware-setup
```

## if Βήμα 13: Σύνδεση σε Wi-Fi
### 1. Σύνδεση σε Wi-Fi
```
nmcli device wifi connect "your_WiFi_Name" password "your_password"
```

## Βήμα 14: Secure Boot Configuration

### 1. Έλεγχος Κατάστασης
```
sbctl status
```

### 2. Δημιουργία και Εγγραφή Κλειδιών
```
sudo sbctl create-keys
sudo sbctl enroll-keys -m
```

### 3. Υπογραφή αρχείων
```
sudo sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi
sudo sbctl sign -s /efi/EFI/BOOT/BOOTX64.EFI
sudo sbctl sign -s /efi/EFI/Linux/arch-linux.efi
sudo sbctl sign -s /efi/EFI/Linux/arch-linux-fallback.efi
```


### 4. Επανεγκατάσταση του πυρήνα
```
sudo pacman -S linux
```

### 5. Επανεκκίνηση
```
sudo reboot
```

## Βήμα 15: Προσθήκη Ανάκαμψης Κλειδιού και TPM

### 1. Εγγραφή Ανάκαμψης Κλειδιού
```
sudo systemd-cryptenroll /dev/gpt-auto-root-luks --recovery-key
```
### 2. Εγγραφή TPM
```
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 /dev/gpt-auto-root-luks
```
