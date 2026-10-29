# 03 — Questions / réponses

Réponses modèles avec justification. À reformuler avec tes mots pour le compte rendu.

## 1. Pourquoi DAD ne permet-il pas de savoir si un routeur est légitime ?

DAD (Duplicate Address Detection) vérifie seulement qu'une adresse n'est pas **déjà utilisée** par un autre hôte avant qu'un hôte ne l'adopte. C'est un contrôle d'**unicité d'adresse**, pas d'identité. DAD n'authentifie personne et ne regarde pas les RA ; il ne peut donc pas distinguer un vrai routeur d'un faux. Un attaquant passe DAD sans problème avec une adresse unique.

## 2. Différence entre `ra-prefixe.conf` et `radvd-prefixe.pid` ?

- `ra-prefixe.conf` : fichier de **configuration** lu par radvd ; il décrit ce qui est annoncé (interface, préfixe, durées). Il définit l'attaque.
- `radvd-prefixe.pid` : fichier technique créé par radvd contenant seulement le **PID** (numéro de processus) de l'instance lancée, pour la repérer et l'arrêter (`kill $(cat /tmp/radvd-prefixe.pid)`).

L'un dit *quoi annoncer*, l'autre dit *quel processus tourne*.

## 3. Pourquoi le port vers R1 reçoit-il la politique RA-ROUTER ?

Parce que R1 est le **routeur légitime**, le seul autorisé à émettre des RA. `device-role router` dit au switch de laisser passer les RA sur ce port. Sans cela, RA Guard bloquerait aussi les RA de R1 et casserait le SLAAC légitime.

## 4. Pourquoi le port vers Kali reçoit-il la politique RA-HOST ?

Parce que Kali est branché comme un **simple hôte**, et qu'un hôte n'a aucune raison légitime d'émettre des RA. `device-role host` bloque tout RA reçu sur ce port → le faux RA de Kali est filtré. Même logique pour le port de la victime (protection si elle est compromise).

## 5. Quelle capture Wireshark prouve que RA Guard protège la victime ?

La capture faite **sur le lien de la victime**, filtre `icmpv6.type == 134`, comparée avant/après RA Guard :

- Le RA de **`FE80::1` (R1) reste présent** → le routeur légitime fonctionne.
- Le RA de **`FE80::66` (Kali) a disparu** → le faux RA n'atteint plus la victime.

C'est cette **disparition du RA de Kali, alors que celui de R1 subsiste**, qui prouve le filtrage. Complément : `ip -6 route` ne montre plus qu'une seule route par défaut (via `FE80::1`).
