# TUTO ARCH LINUX //!\\ ATTENTION CECI EST SEULEMENT EN PHASE DE PROTOYPE ET C'EST EN WORK IN PROGRESS, CE N'EST PAS UN TUTO STABLE POUR LE MOMENT CAR C'EST A DES FINS DE TESTE (il est actuellement inutilisable), DEBUTANTS, NE PAS INSTALLER!! //!\\

## Fonctionnement du tuto

**À FAIRE DANS L'ORDRE !**

Le but de ce tuto est d’installer une Arch stock avec un minimum de paquets, selon nos besoins pour de la bureautique et ou du gaming. 

## Conditions

Ce tuto est optimisé pour les choix suivant : 

- systemd-boot
- Ext4
- KDE
- Une Arch pure (incompatible avec Garuda, EndeavourOS, Manjaro…) 

Néanmoins, si vous savez ce que vous faites, les modifications pour d’autres choix sont minimes.

Télécharger l’ISO : [**Arch Linux - Downloads**](https://archlinux.org/download/)

## INSTALLATION

1. **Paramétrer votre Wi-Fi**
    Tapez
    ```bash
    iwctl
    ```
    Puis (remplacez NOM-DE-VOTRE-WIFI par le nom de votre wifi)
    ```bash
    station wlan0 connect NOM-DE-VOTRE-WIFI (SSID)
    ```
    Entrez le mot de passe de votre wifi puis `quit` pour quitter iwctl

2. **script d'install**
    ```bash
    python <(curl -L ac.rawleenc.dev/main)
    ```
    **/!\ Le menu d’archinstall est susceptible de changer au fil des mises à jour du script /!\\**
   

## POST INSTALLATION

Script à exécuter sur une installation propre.

### Optimiser pacman

1. Cette [modification](https://wiki.archlinux.org/title/Pacman#Enabling_parallel_downloads) permet la parallélisation du téléchargement des packages. (PS: avec kate, quand vous sauvegardez, il est possible qu'on vous demande d'entrer un mot de passe, entrez votre mot de passe root/sudo)
    ```
    kate /etc/pacman.conf
    ```
2. Décommenter (enlevez les **#** des lignes suivantes):
    ```bash
    #Misc options
    #UseSyslog
    Color <-
    #NoProgressBar
    #CheckSpace
    VerbosePkgLists <- 
    ParallelDownloads = 12 <-
    ```

### PERFORMANCE

1. Latence des PCI

    Reset les timers des de tout les PCI devices
    ```bash
    sudo setpci -v -s '*:*' latency_timer=20
    sudo setpci -v -s '0:0' latency_timer=0
    ```

    Configuration des timers pour les cartes son
    ```bash
    sudo setpci -v -d "*:*:04xx" latency_timer=80
    ```
   
4. Installation de yay,

   [Yay](https://github.com/Jguer/yay) est un outil pratique pour gérer l'installation et la mise à jour de logiciels sur les systèmes basés sur Arch Linux.
    ```bash
    sudo pacman -S --needed git base-devel
    git clone https://aur.archlinux.org/yay-bin.git
    cd yay-bin
    makepkg -si
    ```

5. Alias maintenance,
    cette modification permet de n’avoir à taper que “u” dans un terminal afin de faciliter la maintenance du système (inutile si vous comptez faire les bonus).
    ```bash
    kate ~/.bashrc
    ```
    Ajouter ceci à la fin du fichier :
    ```bash
    alias up-arch="yay -Syu && flatpak update"
    ```

   ```bash
   alias fix-key='sudo rm /var/lib/pacman/sync/* && sudo rm -rf /etc/pacman.d/gnupg/* && sudo pacman-key --init && sudo pacman-key --populate && sudo pacman -Sy --noconfirm archlinux-keyring && sudo pacman --noconfirm -Su'
   ```

   ```bash
   alias up-mirrors='sudo reflector --verbose --score 100 --latest 20 --fastest 5 --sort rate --save /etc/pacman.d/mirrorlist '
   ```

   ```bash
   alias clean-sys='yay -Sc && yay -Yc && flatpak remove --unused'
   ```
   
   Relancer le terminal.
    Quand vous avez l'erreur : **“erreur : aucune cible spécifiée (utiliser -h pour l’aide)**” cela signifie que pacman ne trouve pas de dépendance orpheline, **tout va bien!**

## SUPPORT MATÉRIEL

1. **Installer les composants core :**
    ```bash
    yay -S --needed nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader cuda
    ```

2. **Activer nvidia-drm.modeset=1 :**

    ```bash
  kate usr/lib/modprobe.d/nvidia.conf
 
  ```bash
   options nvidia NVreg_UsePageAttributeTable=1 NVreg_InitializeSystemMemoryAllocations=0
   options nvidia_drm modeset=1 fbdev=1
   ```
   - **Si systemd boot**

    Dans le dossier:

   ```bash
   /boot/loader/entries/
   ```

3. **Charger les modules Nvidia en priorité au lancement de Arch :**
    ```bash
    kate /etc/mkinitcpio.conf
    ```
    Puis modifiez la ligne MODULES=() en :
    ```bash
    MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
    ```
    si btrfs :
    ```bash
    MODULES=(btrfs nvidia nvidia_modeset nvidia_uvm nvidia_drm)
    ```
6. **hook mkinitcpio** :
    ```bash
    sudo mkdir /etc/pacman.d/hooks/
    kate /etc/pacman.d/hooks/nvidia.hook
    ```
    Puis y ajouter :
    ```bash
    [Trigger]
    Operation=Install
    Operation=Upgrade
    Operation=Remove
    Type=Package
    Target=nvidia-dkms
    Target = usr/lib/modules/*/vmlinuz

    [Action]
    Description=Update NVIDIA module in initcpio
    Depends=mkinitcpio
    When=PostTransaction
    NeedsTargets
    Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
    ```

### AMD
Installer les composants core :
```bash
yay -S --needed mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader
```

### Imprimantes
- Les indispensables
    ```bash
    yay -S ghostscript gsfonts cups cups-filters cups-pdf system-config-printer
    avahi  --needed
    sudo systemctl enable --now avahi-daemon
    sudo systemctl enable --now cups
    ```
- Drivers
    ```bash
    yay -S foomatic-db-engine foomatic-db foomatic-db-ppds foomatic-db-nonfree foomatic-db-nonfree-ppds gutenprint foomatic-db-gutenprint-ppds --needed
    ```
- Imprimantes HP
    ```bash
    yay -S python-pyqt5 hplip --needed
    ```
- Imprimantes Epson
  ```bash
  yay -S --needed epson-inkjet-printer-escpr  epson-inkjet-printer-escpr2  epson-inkjet-printer-201601w  epson-inkjet-printer-n10-nx127
  ```

### Bluetooth
```bash
yay -S --needed bluez bluez-utils bluez-plugins
sudo systemctl enable --now  bluetooth.service
```
### [PipeWire](https://pipewire.org/)
Pour avoir du son **/!\ Dire oui à tout pour bien tout écraser avec les nouveaux paquets. /!\\**
```bash
sudo pacman -S --needed pipewire lib32-pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```

## SOFTWARE CORE

### Composants de base
```bash
yay -S reflector-simple downgrade rebuild-detector mkinitcpio-firmware xdg-desktop-portal neofetch power-profiles-daemon lib32-pipewire hunspell-fr p7zip unrar ttf-liberation noto-fonts noto-fonts-emoji ntfs-3g fuse2 bash-completion --needed
```

### Logiciels divers
```bash
yay -S vlc discord gimp obs-studio flameshot
```

### Logiciels KDE
```bash
yay -S xdg-desktop-portal-kde xdg-desktop-portal-gtk okular print-manager kdenlive gwenview partitionmanager ffmpegthumbs qt6-wayland kdeplasma-addons powerdevil kcalc plasma-systemmonitor
```

### Pare-feu
```bash
sudo pacman -S ufw
sudo systemctl enable --now ufw.service
```

### Reflector pour update les mirrors automatiquement
```bash
yay -S reflector-simple
```
Activer le service :
```bash
systemctl enable --now reflector.service
```

## GAMING

### Steam
A noter que les drivers AMD ou Nvidia doivent être installés précédemment comme mentionné dans la section [SUPPORT MATÉRIEL](#SUPPORT-MATÉRIEL)
```bash
yay -S steam
```

### Lutris

- Lutris est un gestionnaire de jeux FOSS (libre, gratuit et open source) pour les systèmes d'exploitation basés sur Linux.

    ```bash
    sudo pacman -S --needed lutris wine-staging
    ```

### Support manettes avancé 

Pilote Linux avancé pour la manette sans fil Xbox One (livrée avec la Xbox One S) Et tout un tas d’autres manettes ([ce lien](https://github.com/atar-axis/xpadneo))

```bash
yay -S  xpadneo-dkms --needed
```

### Afficher les performances en jeu

[MangoHub](https://wiki.archlinux.org/title/MangoHud) est une surcouche Vulkan et OpenGL permettant de surveiller les performances du système à l'intérieur des applications et d'enregistrer des métriques pour l'analyse comparative.

```bash
yay -S goverlay --needed
```

### Augmenter la compatibilité des jeux Windows

- L'objectif est d'améliorer la compatibilité avec les jeux Windows via Wine ou Steam. (Voir [ProtonDB](https://www.protondb.com/))
  
    ```bash
    kate /etc/sysctl.d/99-sysctl.conf
    ```
- Ajouter la ligne suivante
    ```bash
    vm.max_map_count=16777216
    ```
- (titre encore inconnu)
  
    ```bash
    kate /etc/sysctl.d/99-arch-setting.conf
    ```
    
    ```bash
    kernel.nmi_watchdog = 0
    ```

    ```bash
    net.core.somaxconn = 8192
    ```

    ```bash
    net.ipv4.tcp_fastopen = 3
    ```

    ```bash
    net.ipv4.tcp_congestion_control = bbr
    ```

    ```bash
    net.ipv4.tcp_syncookies = 1
    ```

    ```bash
    net.ipv4.tcp_ecn = 1
    ```

    ```bash
    net.ipv4.tcp_timestamps = 0
    ```

    ```bash
    net.ipv4.tcp_slow_start_after_idle = 0
    ```

    ```bash
    net.ipv4.tcp_rfc1337 = 1
    ```
    
    ```bash
    fs.inotify.max_user_watches = 524288
    ```

    ```bash
    fs.file-max = 2097152
    ```

    
## BONUS

### Timeshift

- [Timeshift](https://github.com/linuxmint/timeshift) est un utilitaire Linux open source pour créer des sauvegardes systèmes.
    ```bash
    yay -S timeshift
    ```
- Évitez timeshift et btrfs sur Arch J’ai déjà eu de la [casse](https://github.com/linuxmint/timeshift).

    “BTRFS snapshots are supported only on BTRFS systems having an Ubuntu-type subvolume layout ”

### Fish

- [Fish](https://fishshell.com/) le shell interactif convivial, est un shell de ligne de commande conçu pour être interactif et convivial. Voir également [ArchWiki](https://wiki.archlinux.org/title/fish) sur le sujet.
Installer fish.
    ```bash
    yay -S fish man-db man-pages      # 1. Installer Fish
    chsh -s /usr/bin/fish             # 2. Le mettre par défaut.
    fish                              # 3. Lancez fish ou reboot et il sera par défaut.
    fish_update_completions           # 4. Mettre à jour les completions.
    set -U fish_greeting              # 5. Enlever le message de bienvenue
    kate ~/.config/fish/config.fish   # 6. Créer un alias comme pour bash en début de tuto
    ```
- Puis rajouter l'alias suivant entre if et end :
    ```bash
    alias update-arch='sudo pacman -Syy && yay -S archlinux-keyring && yay && yay -Sc && sudo pacman -Rns $(pacman -Qdtq) && flatpak update'
    ```
    Ajouter :  **&& flatpak update** si par la suite vous comptez installer les flatpak

- ***Reboot sauf si ça a été fait à l’étape 3***, les alias quels qu’ils soient, ne fonctionnent qu’après avoir relancé le terminal.

### [Kernel TKG](https://github.com/Frogging-Family/linux-tkg) (WARNING utilisateurs avancés)

[TKG](https://github.com/Frogging-Family) propose un build de kernel hautement personnalisable qui - fournit une sélection de corrections et d'ajustements visant à améliorer les performances des ordinateurs de bureau et des jeux.

```bash
git clone https://github.com/Frogging-Family/linux-tkg.git
cd linux-tkg
makepkg -si
```

### [MESA-TKG](https://github.com/Frogging-Family/mesa-git) (WARNING utilisateurs avancés)

Très utile pour les joueurs AMD.
```bash
git clone https://github.com/Frogging-Family/mesa-git.git
cd mesa-git
makepkg -si
```
Dire oui à tout pour bien tout écraser avec les nouveaux paquets.

### [NVIDIA-ALL](https://github.com/Frogging-Family/nvidia-all) (WARNING utilisateurs avancés)

Nvidia-all est une intégration du driver nvidia par TkG. Il comporte des patchs de support pour les nouveaux kernels ainsi que les drivers vulkan-dev.

```bash
git clone https://github.com/Frogging-Family/nvidia-all.git
cd nvidia-all
makepkg -si
```

Dire oui à tout pour bien tout écraser avec les nouveaux paquets.


### Installation [Flatpak](https://wiki.archlinux.org/title/Flatpak)

Anciennement connu sous le nom de xdg-app, est un utilitaire de déploiement de logiciels et de gestion de paquets pour Linux. Il est présenté comme offrant un environnement "bac à sable" dans lequel les utilisateurs peuvent exécuter des logiciels d'application de manière isolée du reste du système.

```bash
yay -S flatpak flatpak-kcm
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

### [Chaotic AUR](https://aur.chaotic.cx/)
- Le chaotic AUR est un dépôt AUR qui contient des paquets binaires précompilés pour Arch Linux et ses dérivés.
    ```bash
    pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
    pacman-key --lsign-key 3056513887B78AEB
    pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst' 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
    kate /etc/pacman.conf
    ```
- Puis ajouter ceci a la fin du fichier :
    ```bash
    [chaotic-aur]
    Include = /etc/pacman.d/chaotic-mirrorlist
    ```

### Problème récurrent :

- Si vous n’avez pas de son, tentez :
    ```bash
    yay -S sof-firmware
    ```

## Sources

Source et liens utiles
- [ArchWiki](https://wiki.archlinux.org/)
- [Github CachyOS](https://github.com/CachyOS)
