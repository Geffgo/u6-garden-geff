---
{"dg-publish":true,"permalink":"/compte-rendu-technique-sauvegarde-et-migration-de-machine-virtuelle-sur-support-externe/","dg-note-properties":{}}
---

# Compte Rendu Technique : Sauvegarde et Migration de Machine Virtuelle sur Support Externe

## 1. Objectif de l'opération

L'objectif de cette manipulation est de réaliser une sauvegarde complète d'une machine virtuelle (**VM Serveur GLPI**) vers un support de stockage amovible (clé USB). La procédure garantit :

- La réduction de l'espace disque utilisé via la **compression**.
- La **compatibilité inter-systèmes** (Linux/Windows/macOS) grâce au format **exFAT**.
- L'**intégrité des données** lors du transfert.

---

## 2. Procédure de Compression (Archivage)

Afin de faciliter le transport et de réduire le temps de copie, le dossier de la VM est transformé en une archive compressée unique.

- **Accès au répertoire :** `cd ~/VirtualBox\ VMs`
- **Action :** Utilisation de l'outil `tar` avec l'algorithme `gzip`.
- **Commande :** `tar -czvf VM_Serveur_GLPI.tar.gz "VM Serveur GLPI"`
- **Analyse des options :**
    - `-c` : Création de l'archive.
    - `-z` : Compression active (réduction de taille).
    - `-v` : Mode verbeux pour suivre la progression.
    - `-f` : Définition du nom de fichier de sortie.

---

## 3. Préparation du Support de Stockage (Clé USB)

Le système de fichiers **exFAT** est privilégié car, contrairement au FAT32, il permet de stocker des fichiers uniques dépassant **4 Go** (cas fréquent des disques virtuels .vdi ou .vmdk).

### A. Identification et Nettoyage

- **Repérage :** Utilisation de `lsblk` pour identifier le nom du périphérique (ex: `/dev/sdb1`).
- **Sécurité :** Démontage systématique avant manipulation pour éviter toute corruption :
    `sudo umount /dev/sdb1`

### B. Formatage

- **Action :** Création du système de fichiers exFAT avec un label personnalisé.
- **Commande :** `sudo mkfs.exfat -n BACKUP_VM /dev/sdb1`

---

## 4. Transfert et Finalisation

Pour assurer la persistance des données, la clé doit être montée manuellement dans l'arborescence Linux.

### A. Montage et Copie

1. **Création du point de montage :** `sudo mkdir -p /mnt/usb`
2. **Liaison matérielle :** `sudo mount /dev/sdb1 /mnt/usb`
3. **Transfert sécurisé :** Utilisation de `rsync` (recommandé pour sa fiabilité) ou `cp`.
    - `rsync -avh VM_Serveur_GLPI.tar.gz /mnt/usb/`

### B. Libération du support

Il est impératif de libérer le point de montage avant le retrait physique de la clé pour forcer l'écriture des derniers caches de données.

- **Commande :** `sudo umount /mnt/usb`

---

## 5. Synthèse des Commandes Utilisées

| **Étape**          | **Commande principale** | **Rôle**                                             |
| ------------------ | ----------------------- | ---------------------------------------------------- |
| **Archivage**      | `tar -czvf`             | Compresse la VM en un seul fichier.                  |
| **Identification** | `lsblk`                 | Localise la clé USB dans le système.                 |
| **Préparation**    | `mkfs.exfat`            | Formate le support pour les gros fichiers.           |
| **Montage**        | `mount`                 | Rend la clé accessible pour l'écriture.              |
| **Copie**          | `rsync -avh`            | Transfère le fichier avec indicateur de progression. |
| **Sortie**         | `umount`                | Sécurise le retrait du périphérique.                 |

---

## 6. Conclusion et Bonnes Pratiques

L'opération a permis d'isoler la VM dans un format portable. Il est conseillé de vérifier régulièrement l'espace disponible avec `df -h` et d'éviter les caractères spéciaux ou les espaces dans les noms de fichiers d'archives pour garantir une compatibilité maximale entre les terminaux.
