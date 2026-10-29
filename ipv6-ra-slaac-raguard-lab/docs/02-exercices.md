# 02 — Déroulé des 10 exercices

Garde Wireshark ouvert sur le lien de la victime pendant tout le TP. Filtre de base : `icmpv6.type == 133 || icmpv6.type == 134`.

## Ex. 1 — Architecture

Vérifier le câblage : R1 ↔ Gi0/1, victime ↔ Gi0/2, Kali ↔ Gi0/3. Voir [`topology/topology.txt`](../topology/topology.txt).

## Ex. 2 — Configurer R1 (fonctionnement normal)

Appliquer [`configs/r1.cfg`](../configs/r1.cfg). Puis vérifier :

```
show ipv6 interface brief
show ipv6 interface GigabitEthernet0/0
show ipv6 route
```

`ipv6 unicast-routing` active IPv6 **et** l'émission automatique des RA.

## Ex. 3 — Victime Ubuntu (SLAAC normal)

Lancer la capture Wireshark **avant** le down/up :

```bash
ip -br link                 # repérer l'interface, ex. enp0s3
sudo sysctl -w net.ipv6.conf.enp0s3.accept_ra=1
sudo ip link set enp0s3 down
sudo ip link set enp0s3 up
ip -6 addr show dev enp0s3
ip -6 route
```

Résultat : la victime renvoie un RS, R1 répond par un RA, la victime se fabrique une adresse `2001:db8:10::…` et une route par défaut vers `FE80::1`. C'est **SLAAC**.

## Ex. 4 — Victime Windows (variante)

```
ipconfig /all
netsh interface ipv6 show interfaces
netsh interface set interface "Ethernet" admin=disable
netsh interface set interface "Ethernet" admin=enable
ipconfig /all
netsh interface ipv6 show route
```

Filtres : `icmpv6.type == 133` (RS) ou `icmpv6.type == 134` (RA).

## Ex. 5 — Préparer Kali

```bash
sudo apt update
sudo apt install radvd tcpdump
sudo ip link set eth0 up
sudo ip -6 address replace fe80::66/64 dev eth0
sudo systemctl stop radvd
```

Adapter `eth0` au nom réel de l'interface Kali reliée à SW1.

## Ex. 6 — Expérience A : faux préfixe

Utiliser [`configs/radvd/ra-prefixe.conf`](../configs/radvd/ra-prefixe.conf) :

```bash
sudo radvd -n -C /tmp/ra-prefixe.conf -p /tmp/radvd-prefixe.pid
```

- `-n` : premier plan (Ctrl+C pour arrêter)
- `-C` : fichier de configuration
- `-p` : fichier PID (numéro de processus, pas une config)

Résultat : la victime reçoit une **2e adresse** en `2001:db8:66::…`. Arrêter avec **Ctrl+C**.

## Ex. 7 — Expérience B : faux routeur par défaut

Utiliser [`configs/radvd/ra-routeur.conf`](../configs/radvd/ra-routeur.conf) :

```bash
sudo radvd -n -C /tmp/ra-routeur.conf -p /tmp/radvd-routeur.pid
```

`AdvDefaultLifetime 60` + `AdvDefaultPreference high` : Kali devient routeur par défaut candidat → interception possible (MITM).

## Ex. 8 — Observer l'effet

| Machine | Commandes |
| --- | --- |
| Ubuntu | `ip -6 addr` ; `ip -6 route` ; `sudo tcpdump -ni enp0s3 'icmp6'` |
| Windows | `ipconfig /all` ; `netsh interface ipv6 show route` ; `route print -6` |
| Wireshark | `icmpv6.type == 134` : comparer `FE80::1` et `FE80::66`, Router Lifetime, Router Preference |

La victime a maintenant **deux routes par défaut** → preuve que le faux RA a marché.

## Ex. 9 — Protéger avec RA Guard

Appliquer [`configs/sw1-raguard.cfg`](../configs/sw1-raguard.cfg) sur SW1.

- `device-role router` sur Gi0/1 → les RA de R1 passent.
- `device-role host` sur Gi0/2 et Gi0/3 → tout RA de la victime ou de Kali est bloqué.

## Ex. 10 — Vérifier le blocage

1. R1 continue d'envoyer ses RA.
2. Relancer `radvd` sur Kali (expérience B).
3. Capturer sur le lien de la victime.
4. Le RA de `FE80::1` **reste visible**.
5. Le RA de `FE80::66` **ne doit plus atteindre** la victime.

```
show ipv6 nd raguard policy
show running-config interface GigabitEthernet0/1
show running-config interface GigabitEthernet0/3
```

| Observation | Résultat attendu |
| --- | --- |
| RA de R1 visible sur la victime | Oui (avant et après) |
| RA de Kali visible avant RA Guard | Oui |
| RA de Kali visible après RA Guard | Non (bloqué) |
| Route par défaut après protection | Uniquement via `FE80::1` |
