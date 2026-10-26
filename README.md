## Preface

Linux a

---

# PARTIE 1 — CONCEPTS FONDAMENTAUX

## 1. Architecture du système

```
APPLICATION       →  navigateur, IDE, etc.
SHELL / CLI       →  bash, zsh, fish
SYSTEM CALL       →  interface entre user-space et kernel-space
KERNEL            →  cœur du système (process, mémoire, drivers)
HARDWARE          →  CPU, RAM, disque
```

### Rôle du Kernel

- **Process management** : créer, planifier (scheduler), tuer des processus
- **Memory management** : gestion RAM, swap, paging
- **File management** : lecture, écriture, organisation des fichiers
- **Drivers management** : communication avec le matériel
- **Network management** : pile TCP/IP, sockets

### Commandes utiles

```bash
uname -r      # version du kernel
uname -a      # infos système complètes
lsmod         # modules chargés
modprobe      # charger un module
dmesg         # messages du kernel (boot, erreurs matérielles)
```

### System calls (à approfondir)

Un **system call** (appel système) est le mécanisme par lequel un programme en espace utilisateur demande un service au kernel (ouvrir un fichier, allouer de la mémoire, créer un processus...).

```bash
strace {cmd}     # trace tous les system calls d'une commande
strace -c {cmd}  # résumé statistique des syscalls utilisés
ltrace {cmd}     # trace les appels aux bibliothèques (lib calls)
```

Exemples de syscalls courants : `open()`, `read()`, `write()`, `fork()`, `exec()`, `exit()`.

---

## 2. Permissions

```
r (4) = read
w (2) = write
x (1) = execute

--- --- ---
 u   g   o     (user / group / other)
```

Exemple : `chmod 754 file`

- `7` = rwx (user)
- `5` = r-x (group)
- `4` = r-- (other)

### Cas particuliers

```
SUID (4000)  # exécute avec les droits du propriétaire du fichier
SGID (2000)  # exécute avec les droits du groupe propriétaire
Sticky (1000) # seul le propriétaire peut supprimer son fichier (ex: /tmp)
```

```bash
chmod u+s file      # SUID
chmod g+s folder     # SGID
chmod +t folder      # sticky bit
```

---

## 3. Filesystem Hierarchy (FHS)

```
/          racine
├── bin/    binaires essentiels (utilisateurs)
├── sbin/   binaires système (admin)
├── etc/    fichiers de configuration
├── home/   dossiers utilisateurs
├── root/   dossier de l'utilisateur root
├── var/    données variables (logs, cache, mail)
├── tmp/    fichiers temporaires
├── usr/    programmes et données partagées
│   ├── bin/
│   ├── lib/
│   └── share/
├── lib/    bibliothèques partagées
├── dev/    fichiers de périphériques
├── proc/   infos processus/kernel (virtuel)
├── sys/    infos kernel/matériel (virtuel)
├── mnt/    points de montage temporaires
├── media/  montage automatique (USB, CD)
├── boot/   fichiers de démarrage (kernel, GRUB)
└── opt/    logiciels tiers optionnels
```

### Systèmes de fichiers

| FS    | Usage                     |
| ----- | ------------------------- |
| ext4  | Standard Linux            |
| xfs   | Haute performance         |
| btrfs | Snapshots, moderne        |
| fat32 | Compatibilité universelle |
| ntfs  | Compatibilité Windows     |
| tmpfs | Stocké en RAM (volatile)  |

---

## 4. Gestion des partitions et disques

```bash
lsblk                  # lister disques/partitions
df -h                  # espace disque utilisé (humain)
du -sh /folder         # taille d'un dossier
fdisk -l               # détails des partitions
mount /dev/sdb1 /mnt   # monter une partition
umount /mnt            # démonter (attention : umount, pas "unmount")
cat /etc/fstab         # montage automatique au boot
```

### Structure des devices

```
/dev/sda        # disque entier (SATA/SCSI)
├── /dev/sda1   # partition 1 (boot)
├── /dev/sda2   # partition 2 (/)
└── /dev/sda3   # partition 3 (swap)

/dev/nvme0n1    # SSD NVMe
├── /dev/nvme0n1p1
└── /dev/nvme0n1p2
```

---

## 5. Processus

### États

|Code|État|
|---|---|
|R|Running (en exécution)|
|S|Sleeping (attente d'événement)|
|D|Disk sleep (attente I/O, non interruptible)|
|Z|Zombie (terminé, non nettoyé par le parent)|
|T|Stopped (en pause)|

### Hiérarchie

```
PID 1 (systemd/init)
└── PID 100 (bash)
    ├── PID 200 (firefox)
    └── PID 300 (vim)
```

### Commandes

```bash
ps aux                 # liste tous les processus
pstree                 # arbre des processus
top / htop             # monitoring temps réel
kill {PID}              # terminer un processus (SIGTERM)
kill -9 {PID}           # forcer (SIGKILL)
killall firefox         # tuer par nom
nice -n 10 {cmd}        # lancer avec une priorité (-20 à 19)
renice 10 {PID}         # changer la priorité d'un processus existant
```

---

## 6. Mémoire

```bash
free -h                 # RAM utilisée/disponible
cat /proc/meminfo       # détails mémoire
vmstat                  # statistiques mémoire/CPU
```

### Types de mémoire

- **RAM** : mémoire physique
- **Swap** : mémoire virtuelle sur disque (extension de la RAM)
- **Buffer** : données en attente d'écriture sur disque
- **Cache** : données fréquemment utilisées gardées en mémoire

---

## 7. Processus de démarrage (Boot)

1. **BIOS / UEFI** → test matériel (POST)
2. **Bootloader (GRUB)** → choix du kernel à charger
3. **Kernel Linux** → initialise le matériel, monte le filesystem racine
4. **Init system (systemd)** → démarre les services et l'environnement
5. **Login / Interface graphique**

```bash
systemd-analyze          # temps de démarrage
systemd-analyze blame     # temps par service
```

---

## 8. Systemd

```bash
systemctl status {service}
systemctl start {service}
systemctl stop {service}
systemctl restart {service}
systemctl enable {service}    # activer au démarrage
systemctl disable {service}
systemctl list-units          # liste des unités actives
systemctl list-unit-files     # liste de tous les services connus
journalctl -u {service}       # logs d'un service
journalctl -f                 # logs en direct
journalctl -b                 # logs depuis le dernier boot
journalctl -p err              # logs filtrés par priorité (erreurs)
```

---

## 9. Signaux

|Signal|Numéro|Effet|
|---|---|---|
|SIGHUP|1|Recharger la configuration|
|SIGKILL|9|Tuer immédiatement (non interceptable)|
|SIGTERM|15|Demande d'arrêt propre|
|SIGSTOP|19|Mettre en pause|
|SIGCONT|18|Reprendre l'exécution|

```bash
kill -15 {PID}    # SIGTERM (par défaut)
kill -9 {PID}     # SIGKILL
kill -1 {PID}     # SIGHUP
kill -l           # liste tous les signaux disponibles
```

---

## 10. Liens

```bash
# Lien symbolique (raccourci, pointe vers un chemin)
ln -s /source /destination
ln -s /var/www/html /home/jhon/site

# Lien physique (hard link, même inode, même fichier réel)
ln /source /destination

# Visualiser les liens
ls -al
readlink file       # destination d'un lien symbolique
stat file           # infos détaillées incluant inode
```

**Différence clé** : un lien symbolique casse si la source est supprimée ; un lien physique pointe vers le même inode et survit à la suppression du nom original (tant qu'un lien existe).

---

## 11. TTY — qu'est-ce que c'est ?

**TTY** (TeleTYpewriter) désigne un terminal, physique ou virtuel, permettant l'interaction avec le système.

```bash
tty                # affiche le terminal courant (ex: /dev/pts/0)
who                # liste les utilisateurs connectés et leur tty
w                  # comme who + activité en cours
```

- `tty1` à `tty6` : consoles virtuelles (Ctrl+Alt+F1-F6)
- `pts/0`, `pts/1`... : pseudo-terminaux (terminal graphique, SSH)

---

## 12. Logs système

Les logs sont essentiels pour diagnostiquer les problèmes.

```bash
journalctl                  # tous les logs systemd
journalctl -u ssh           # logs d'un service précis
journalctl -f               # suivi en direct (comme tail -f)
journalctl -b               # logs du boot actuel
journalctl -b -1            # logs du boot précédent
journalctl --since "1 hour ago"
journalctl -p err..alert    # filtrage par niveau de gravité
```

### Fichiers de logs classiques (systèmes sans journald pur)

```
/var/log/syslog       # logs généraux (Debian/Ubuntu)
/var/log/auth.log     # authentification, sudo, ssh
/var/log/kern.log     # messages du kernel
/var/log/dmesg        # buffer de messages kernel au boot
/var/log/boot.log     # démarrage
/var/log/apt/         # historique des installations de paquets
```

```bash
tail -f /var/log/syslog     # suivre un log en direct
grep "error" /var/log/syslog # rechercher des erreurs
```

---

# PARTIE 2 — COMMANDES ESSENTIELLES

## 1. Navigation et fichiers

```bash

pwd                          # position actuelle (Print Working Directory)
cd {destination}             # se déplacer
cd ..                        # remonter d'un niveau
cd ~                         # aller au home
ls                           # lister le contenu
ls -la                       # lister avec détails et fichiers cachés
mkdir {dossier}              # créer un dossier
mkdir -p a/b/c               # créer une arborescence complète
touch {fichier}              # créer un fichier vide / mettre à jour la date
cp {source} {dest}           # copier
cp -r {source} {dest}        # copier récursivement (dossiers)
mv {source} {dest}           # déplacer ou renommer

rm {fichier}                 # supprimer un fichier
rm -rf {dossier}             # supprimer un dossier (récursif, forcé)
rmdir {dossier}              # supprimer un dossier vide
```

## 2. Lecture et écriture de fichiers

```bash
cat {file}          # afficher tout le fichier
less {file}         # navigation page par page (recommandé pour gros fichiers)
head {file}         # 10 premières lignes
head -n 20 {file}   # 20 premières lignes
tail {file}         # 10 dernières lignes
tail -f {file}      # suivre en direct (logs)
nano {file}         # éditeur simple par défaut
vim {file}          # éditeur avancé modal
```

> Note corrigée : `less` sert à naviguer dans un fichier page par page (et pas à afficher les 10 dernières lignes, c'est `tail`).

## 3. Permissions

```bash
chmod +x file        # ajouter droit d'exécution
chmod 755 file        # définir les droits en octal
chown user file       # changer le propriétaire
chown user:group file # changer propriétaire + groupe
sudo {cmd}            # exécuter en super-utilisateur
```

## 4. Gestion des paquets (Debian/Ubuntu — APT)

```bash
apt update                  # mettre à jour la liste des paquets disponibles
apt upgrade                 # mettre à jour les paquets installés
apt install {package}       # installer un paquet
apt remove {package}        # supprimer un paquet (garde la config)
apt purge {package}         # supprimer paquet + configuration
apt autoremove              # supprimer les dépendances inutilisées
apt list --installed        # liste des paquets installés
apt search {mot}            # rechercher un paquet
```

Autres gestionnaires selon distribution :

```bash
dnf install {package}   # Fedora/RHEL
yum install {package}   # anciennes versions RHEL
pacman -S {package}     # Arch Linux
```

## 5. Recherche

```bash
find / -name {filename}       # recherche par nom dans le filesystem
find / -type f -mtime -1      # fichiers modifiés dans les dernières 24h
grep {mot} {file}              # rechercher un mot dans un fichier
grep -r {mot} /folder          # recherche récursive dans un dossier
grep -i {mot} {file}           # recherche insensible à la casse
grep -n {mot} {file}           # afficher les numéros de ligne
locate {file/folder}           # recherche instantanée via base de données
updatedb                       # mettre à jour la base de locate
which {cmd}                    # chemin d'un exécutable
whereis {cmd}                  # chemin binaire + man + source
```

## 6. Processus

```bash
ps aux              # liste des processus en cours
top                  # monitoring temps réel
htop                 # version améliorée de top (si installé)
kill {PID}           # terminer un processus
kill -9 {PID}        # forcer la terminaison
```

## 7. Réseau

```bash
ping {host}                    # tester la connectivité
ip a                            # afficher les interfaces réseau
ip route                        # table de routage
curl {url}                      # requête web / téléchargement
curl -O {url}                   # télécharger en gardant le nom du fichier
wget {url}                      # télécharger un fichier
ssh user@host                   # connexion à distance
scp file user@host:/path        # copier un fichier via SSH
netstat -tulnp                  # ports ouverts (ou `ss -tulnp`)
```

## 8. Aide

```bash
help                # aide sur les commandes internes du shell
man {cmd}            # documentation complète d'une commande
{cmd} --help         # aide rapide
whatis {cmd}         # description en une ligne
```

---

# PARTIE 3 — GESTION DES UTILISATEURS

## 1. Commandes de gestion

```bash
useradd {user}              # créer un utilisateur
useradd -m {user}           # créer avec un dossier home
passwd {user}                # définir/changer le mot de passe
userdel {user}                # supprimer un utilisateur
userdel -r {user}             # supprimer + son dossier home
usermod -aG {group} {user}    # ajouter un utilisateur à un groupe

groupadd {group}              # créer un groupe
groupdel {group}               # supprimer un groupe
groupmod -n {new} {old}        # renommer un groupe
gpasswd -a {user} {group}      # ajouter un utilisateur à un groupe

groups {user}                  # liste des groupes d'un utilisateur
id {user}                       # UID, GID, groupes
```

## 2. Fichiers de configuration système

```bash
cat /etc/passwd     # infos utilisateurs
cat /etc/shadow     # mots de passe chiffrés (root uniquement)
cat /etc/group      # infos groupes
cat /etc/sudoers    # règles sudo
sudo visudo         # éditer sudoers en sécurité (vérifie la syntaxe)
```

### Format de `/etc/passwd`

```
username:password:UID:GID:comment:home:shell
jhon:x:1001:1001:John Doe:/home/jhon:/bin/bash
```

- `password` = `x` signifie que le mot de passe réel est dans `/etc/shadow`
- `UID` = identifiant utilisateur (0 = root)
- `GID` = identifiant du groupe principal

## 3. Informations utilisateur/session

```bash
whoami          # utilisateur courant
id              # UID/GID/groupes
who             # utilisateurs connectés
w               # utilisateurs connectés + activité
last            # historique des connexions
lastlog         # dernière connexion de chaque utilisateur
```

## 4. Sudo et changement d'utilisateur

```bash
sudo {cmd}                    # exécuter une commande en root
sudo -u {user} {cmd}          # exécuter en tant qu'un autre utilisateur
sudo su                        # devenir root (session complète)
su {user}                      # basculer vers un autre utilisateur (demande mdp)
su -                            # devenir root avec son environnement complet
```

---

# PARTIE 4 — COMMANDS BUILDINGS

### File descriptor (fd)

Input -> What you give to terminal.
Output -> What your terminal return.

```
0        STDIN         Standard input
1        STDOUT        Standard output
2        STDERR        Error output
```


Exemple : 
```bash
$ cat file.txt > copy.txt
Hello, world !
$
```
Here the output code is 1 (STDOUT), if the file not existing, the error will be 2 (STDERR) and our command have the code 0 (STDIN)


### Tout les opérateurs


| Opérateur / symbole | Fonction                                                 |
| ------------------- | -------------------------------------------------------- |
| `>`                 | Rediriger STDOUT en écrasant le fichier                  |
| `>>`                | Rediriger STDOUT en ajoutant au fichier                  |
| `1>`                | Rediriger explicitement STDOUT                           |
| `2>`                | Rediriger STDERR                                         |
| `2>&1`              | Envoyer STDERR vers la destination actuelle de STDOUT    |
| `&>`                | Rediriger STDOUT et STDERR vers le même fichier (Bash)   |
| `<`                 | Fournir un fichier comme STDIN                           |
| `\|`                | Connecter STDOUT à STDIN de la commande suivante         |
| `\|&`               | Connecter STDOUT et STDERR à la commande suivante (Bash) |
| `&&`                | Exécuter la suite si la commande réussit                 |
| `\|\|`              | Exécuter la suite si la commande échoue                  |
| `;`                 | Exécuter la commande suivante sans condition             |
| `&`                 | Lancer la commande en arrière-plan                       |
| `$()`               | Substituer la sortie d'une commande                      |
| ``...``             | Ancienne syntaxe de substitution de commandes            |
| `{ ...; }`          | Grouper dans le shell courant                            |
| `( ... )`           | Grouper dans un sous-shell                               |
| `tee`               | Dupliquer STDIN vers STDOUT et un fichier                |
| `xargs`             | Construire des commandes à partir de STDIN               |
(Rajouter des exemples concret)


### Grep


# PARTIE 5 — CRON
# PARTIE 6 — SYSTEM CALLS


# À approfondir / compléter

- [ ]  **System calls** en détail (table des syscalls, `strace` avancé)
- [ ]  **grep avancé** (expressions régulières, `-E`, `-P`, `sed`, `awk`)
- [ ]  **Cron / tâches planifiées** (`crontab`, `at`, `systemd timers`)