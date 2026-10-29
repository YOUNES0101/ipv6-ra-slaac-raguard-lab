# TP IPv6 — Router Advertisements, SLAAC, faux RA & RA Guard

Laboratoire de sécurité réseau reproduisant une attaque par **faux Router Advertisement (rogue RA)** sur IPv6, puis sa mitigation par **RA Guard**. Réalisé entièrement dans un environnement **GNS3 / VirtualBox isolé**.

> Module : *Protocoles de sécurité et cryptanalyse* — EMSI

## Objectif

1. Comprendre l'autoconfiguration IPv6 normale : `RS → RA → SLAAC`.
2. Démontrer, en labo contrôlé, comment un attaquant détourne ce mécanisme avec un faux RA.
3. Protéger le réseau local avec **RA Guard** (IPv6 First-Hop Security) et prouver le blocage.

## Dans quelle couche ?

| Élément | Couche OSI |
| --- | --- |
| IPv6, ICMPv6, messages RS/RA (NDP) | Couche 3 (réseau) |
| Attaque faux RA | Couche 3 (réseau) |
| RA Guard (switch) | Appliqué en couche 2, inspecte du contenu de couche 3 |

## Topologie

```
            R1  (FE80::1  /  2001:DB8:10::1/64)
                     |  Gi0/0 de R1  <->  Gi0/1 de SW1
                 +---+----+
                 |  SW1   |   <- RA Guard appliqué ici
                 +--+--+--+
          Gi0/2  |     |  Gi0/3
                 |     |
             Victime   Kali  (FE80::66)
         (SLAAC auto)  (faux RA via radvd)
```

| Machine | Rôle | OS / image |
| --- | --- | --- |
| R1 | Routeur légitime, annonce `2001:DB8:10::/64` | Cisco IOSv / IOU / Dynamips |
| SW1 | Switch, applique RA Guard | Cisco IOS/IOS XE compatible First-Hop Security |
| Victime | Client auto-configuré par SLAAC | Ubuntu ou Windows (VM VirtualBox) |
| Kali | Émet le faux RA | Kali Linux (VM VirtualBox) |

## Structure du dépôt

```
.
├── README.md
├── docs/
│   ├── 00-fondations.md          # Comment IPv6 fonctionne (bases, NDP, couches)
│   ├── 01-installation.md        # Installer GNS3, VirtualBox, Kali, la victime
│   ├── 02-exercices.md           # Déroulé des 10 exercices
│   └── 03-questions-reponses.md  # Réponses justifiées aux 5 questions
├── configs/
│   ├── r1.cfg                    # Configuration du routeur légitime
│   ├── sw1-raguard.cfg           # Configuration RA Guard du switch
│   └── radvd/
│       ├── ra-prefixe.conf       # Expérience A : faux préfixe
│       └── ra-routeur.conf       # Expérience B : faux routeur par défaut
└── topology/
    └── topology.txt              # Schéma de la topologie
```

## Démarrage rapide

1. Monter l'environnement : voir [`docs/01-installation.md`](docs/01-installation.md).
2. Appliquer la config de R1 : [`configs/r1.cfg`](configs/r1.cfg).
3. Observer le SLAAC normal, puis lancer l'attaque depuis Kali avec les fichiers de [`configs/radvd/`](configs/radvd).
4. Appliquer RA Guard : [`configs/sw1-raguard.cfg`](configs/sw1-raguard.cfg).
5. Vérifier le blocage (Wireshark, filtre `icmpv6.type == 134`).

## Filtres Wireshark utiles

| Filtre | Montre |
| --- | --- |
| `icmpv6.type == 133` | RS seuls |
| `icmpv6.type == 134` | RA seuls |
| `icmpv6.type == 133 || icmpv6.type == 134` | RS et RA |

## Avertissement

Ce laboratoire injecte de faux Router Advertisements. Il doit **uniquement** être exécuté dans l'environnement GNS3/VirtualBox isolé décrit ici. Émettre de faux RA sur un réseau réel ou partagé est illégal et dangereux.

## Licence

Usage pédagogique. À adapter selon les règles de ton établissement.
