# 01 — Installation de l'environnement

GNS3 dessine la topologie mais délègue le travail à la **GNS3 VM**, hébergée dans VirtualBox. Ordre : VirtualBox → GNS3 → GNS3 VM → Kali & victime → images Cisco → câblage.

## 0. Pré-requis

- Windows 10/11 64 bits, 8 Go de RAM mini (16 Go confortable), ~40 Go libres.
- **Virtualisation activée dans le BIOS/UEFI** (Intel VT-x / AMD-V). Vérifier : Gestionnaire des tâches → Performance → CPU → « Virtualisation : Activé ».
- **Désactiver Hyper-V** et « Plateforme de machine virtuelle » (conflit avec VirtualBox).

## 1. VirtualBox

Installer **Oracle VirtualBox 7.0.x** (Windows hosts) depuis virtualbox.org, options par défaut, + Extension Pack.

## 2. GNS3 (interface)

1. Télécharger GNS3 pour Windows (compte communautaire GNS3 gratuit requis).
2. Installer avec les composants par défaut (**Npcap + Wireshark** inclus — à garder).
3. Mettre le dossier GNS3 en exception de l'antivirus (`gns3server`, `ubridge`, `dynamips`).

## 3. GNS3 VM

1. Télécharger l'image **GNS3 VM pour VirtualBox** — **même version que le GNS3 installé**. <https://www.gns3.com/software/download-vm>
2. Dézipper, puis dans VirtualBox : **Fichier → Importer un appareil virtuel** → le `.ova` → Importer.
3. **Ne pas démarrer la VM à la main.** Lancer GNS3 ; dans l'assistant, choisir d'exécuter les appliances sur la **GNS3 VM**.
4. GNS3 cherche VMware par défaut et affiche une erreur → OK → **choisir VirtualBox**. La GNS3 VM apparaît alors. Next → Finish.
5. Le voyant de la GNS3 VM dans GNS3 doit passer au **vert**.

> Routeurs qui ne bootent jamais (code qui défile) → désactiver KVM dans les réglages de la GNS3 VM.

## 4. Kali Linux (attaquant)

1. Importer l'image **Kali pour VirtualBox** (OVA) depuis kali.org.
2. Carte réseau en **NAT** temporairement, démarrer, installer les outils :
   ```bash
   sudo apt update
   sudo apt install radvd tcpdump
   ```
3. Éteindre, remettre la carte en **Generic Driver (UDP Tunnel)**.
4. Intégrer à GNS3 : **Edit → Preferences → VirtualBox VMs → New** → VM Kali → Finish.

## 5. Victime (Ubuntu ou Windows)

- Créer une VM Ubuntu Desktop ou Windows dans VirtualBox, l'intégrer à GNS3 comme Kali (carte en Generic Driver).
- Noter le nom de l'interface : Ubuntu `ip -br link` (souvent `enp0s3`) ; Windows → « Ethernet ».

## 6. Routeur R1 et switch SW1

- **File → New template → Install an appliance from the GNS3 server.**
- **R1** : image Cisco supportant IPv6 (IOSv / c7200 / IOU). Les images Cisco ne sont pas fournies par GNS3 → utiliser celle **fournie par l'établissement** ou sous licence.
- **SW1** : image supportant **IPv6 First-Hop Security** (`ipv6 nd raguard`), type **IOSvL2 / IOU L2** récent. Point critique : sans cette fonction, RA Guard ne marchera pas.

## 7. Câblage

| Lien | De | Vers |
| --- | --- | --- |
| R1 ↔ SW1 | Gi0/0 de R1 | Gi0/1 de SW1 |
| Victime ↔ SW1 | carte victime | Gi0/2 de SW1 |
| Kali ↔ SW1 | carte Kali | Gi0/3 de SW1 |

Démarrer tous les nœuds. Tout est isolé du réseau réel — c'est voulu.

*Versions à jour au 29 octobre 2026. VirtualBox 7.0.x ; GNS3 et sa VM doivent avoir la même version.*
