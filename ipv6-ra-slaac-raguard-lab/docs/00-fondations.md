# 00 — Fondations : comment IPv6 fonctionne

Avant de manipuler l'attaque, il faut comprendre le terrain. Cette page pose les bases.

## 1. Les couches concernées

Rappel du modèle OSI : couche 1 physique, couche 2 liaison (switch, MAC), couche 3 réseau (IP, routage), couche 4 transport (TCP/UDP).

- **IPv6** = couche 3 (réseau).
- Les messages **RS / RA** voyagent dans **ICMPv6**, porté par IPv6 → couche 3.
- L'**attaque du faux RA** est donc une attaque de **couche 3** : on ment sur l'autoconfiguration réseau.
- **RA Guard** est appliqué sur le **switch (couche 2)**, mais il inspecte le contenu de couche 3 (les RA) pour les filtrer. C'est le principe du *First-Hop Security* : un équipement L2 qui regarde dans le L3.

> À retenir : **attaque en couche 3, protection posée au niveau de la couche 2.**

## 2. Les adresses IPv6

- **128 bits** (contre 32 en IPv4), écrites en hexadécimal, en 8 blocs de 16 bits.
- Exemple : `2001:0db8:0010:0000:0000:0000:0000:0001`, compressé en `2001:db8:10::1`.
- **Pas de broadcast** en IPv6.

| Type | Plage | Rôle |
| --- | --- | --- |
| Link-local | `FE80::/10` | valable seulement sur le lien local ; chaque interface IPv6 en a une |
| Globale unicast | `2000::/3` (ex. `2001:db8:10::/64`) | routable sur Internet |
| Multicast | `FF00::/8` | remplace le broadcast |

- Un réseau = un préfixe **/64** : les 64 premiers bits = le réseau, les 64 derniers = l'identifiant d'interface.

## 3. NDP — Neighbor Discovery Protocol

NDP est le protocole central d'IPv6. Il **remplace l'ARP** d'IPv4 et gère la découverte des routeurs et des voisins. Il s'appuie sur **5 messages ICMPv6** :

| Type | Message | Rôle |
| --- | --- | --- |
| 133 | RS — Router Solicitation | l'hôte cherche un routeur |
| 134 | RA — Router Advertisement | le routeur s'annonce + préfixe + route par défaut |
| 135 | NS — Neighbor Solicitation | « qui possède cette adresse ? » (rôle de l'ARP) |
| 136 | NA — Neighbor Advertisement | réponse à un NS |
| 137 | Redirect | le routeur signale un meilleur chemin |

## 4. Obtenir une adresse : 3 méthodes

| Méthode | Principe | Serveur requis ? |
| --- | --- | --- |
| **SLAAC** | l'hôte se fabrique son adresse à partir du préfixe reçu dans un RA | Non |
| **DHCPv6** | un serveur distribue les adresses (comme le DHCP IPv4) | Oui |
| **Statique** | adresse configurée manuellement | Non |

### Le déroulé de SLAAC (le mécanisme attaqué)

1. L'hôte démarre et envoie un **RS** (« y a-t-il un routeur ? »).
2. Le routeur répond par un **RA** (« voici le préfixe `2001:db8:10::/64` et je suis ta route par défaut »).
3. L'hôte prend le préfixe, ajoute un identifiant d'interface, et obtient son adresse globale — sans serveur.
4. Avant d'adopter l'adresse, l'hôte vérifie qu'elle est unique avec **DAD** (Duplicate Address Detection, via NS/NA).

## 5. Ces protocoles sont-ils toujours utilisés ?

- **NDP (RS / RA / NS / NA) : oui, toujours.** C'est le cœur d'IPv6. Aucun LAN IPv6 ne fonctionne sans, car l'ARP n'existe pas en IPv6.
- **SLAAC : pas toujours.** C'est la méthode par défaut la plus courante, mais DHCPv6 ou le statique sont des alternatives. Même alors, les **RA restent présents** : ils indiquent aux hôtes quoi faire via les drapeaux **M** (Managed, = utilise DHCPv6 pour l'adresse) et **O** (Other, = utilise DHCPv6 pour les options comme le DNS).

**Conséquence sécurité :** comme les RA sont quasi universels en IPv6 et qu'aucun mécanisme ne les authentifie par défaut, le faux RA est une menace réelle et générale — d'où l'existence d'une protection standard : RA Guard.
