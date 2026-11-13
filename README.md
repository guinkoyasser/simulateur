# Simulateur du lab Nmap

Simulateur pédagogique en page web unique (`index (1).html`) qui rejoue un vrai laboratoire de reconnaissance réseau : deux machines virtuelles (**Kali Linux**, l'analyse, et **Ubuntu Server**, la cible) sur un réseau isolé `192.168.86.0/24`. Toutes les commandes et sorties sont simulées — rien n'est réellement scanné, tout est calculé dans la page — mais les sorties reproduisent des mesures observées sur un vrai lab VMware (Kali, Ubuntu Server, Nmap 7.99).

Le jeu fonctionne comme un **guide pas-à-pas** : chaque étape affiche un objectif concret avant la commande à taper, les prérequis bloquants sont annoncés à l'avance, et la progression (étapes faites, commandes exécutées) se coche au fur et à mesure.

## Lancer le simulateur

Aucune installation ni serveur requis : ouvrez `index (1).html` directement dans un navigateur.

## Déroulement en deux parties

| Partie | Pare-feu | Objectif |
|---|---|---|
| **Partie 1** (14 étapes) | désactivé | Cartographier entièrement la cible sans restriction réseau, et apprendre à lire un scan Nmap. |
| **Partie 2** (5 étapes) | actif | Refaire les mêmes vérifications avec le pare-feu actif, pour mesurer précisément son effet sur chaque port et dans les journaux. |

Une étape d'introduction et un bilan final encadrent les deux parties (21 étapes au total).

### Partie 1 — Pare-feu désactivé
1. Trouver l'adresse IP de la cible (`ip addr`)
2. Vérifier l'état du pare-feu (`sudo ufw status`)
3. Voir les services qui écoutent (`ss -tln`)
4. Passer sur Kali et relever son adresse (`ip addr`)
5. Tester la connectivité par ping (`ping -c 4 ...`)
6. Vérifier Nmap (`nmap --version`)
7. Premier scan de la cible (`nmap ...`)
8. Scanner tous les ports (`nmap -n -p- ...`)
9. Détecter le logiciel et sa version (`nmap -n -sV -p 22 ...`)
10. Découvrir toutes les machines actives (`nmap -sn .../24`)
11. Installer un deuxième service — Apache (`sudo apt install -y apache2`, carte NAT requise)
12. Voir l'effet d'un nouveau service (`nmap -n -p 22,80,443 ...`)
13. Identifier Apache et voir sa trace (`nmap -n -sV -p 22,80 ...`)
14. Voir la trace du scan dans les journaux (`sudo tail -n 5 /var/log/apache2/access.log`)

### Partie 2 — Pare-feu actif
15. Activer le pare-feu et bloquer le port 80 (séquence : `ufw allow 22/tcp`, `ufw deny 80/tcp`, `ufw enable`)
16. Confirmer la nouvelle configuration (`sudo ufw status verbose`)
17. Scanner avec le pare-feu actif — même scan qu'en partie 1, pour comparer
18. Lire les traces du pare-feu (`sudo tail -n 5 /var/log/ufw.log`)
19. Vérifier qu'Apache tourne toujours malgré le filtrage (`ss -tln`)

## Fonctionnalités du guide

- **Pastille de partie** toujours visible (verte = pare-feu désactivé, orange = pare-feu actif) avec le compte d'étapes de la partie en cours.
- **Bannière d'introduction** à l'entrée de chaque partie, rappelant son objectif global.
- **Bloc Objectif** sur chaque étape : ce qu'il faut accomplir, avant même de lire la théorie.
- **Bloc Prérequis** quand une étape dépend d'une action préalable (ex. activer la carte NAT avant d'installer Apache) — annoncé à l'avance, jamais découvert par l'échec.
- **Checklist à cocher** pour les étapes à plusieurs commandes.
- **Analyse structurée** après chaque commande réussie : *Ce que vous voyez* / *Pourquoi* / *À retenir*.
- **Deux machines** avec terminaux et historiques séparés (onglets en haut), bascule automatique vers la bonne machine à chaque étape.
- **Panneau de contrôle de la cible** (à droite) : services actifs, règles de pare-feu, carte NAT.
- **Progression sauvegardée** dans `localStorage` du navigateur (clé `nmaplab-sim-v1`) — reprise proposée à la prochaine ouverture.
- **Bilingue FR/EN**, bascule instantanée via le sélecteur en haut à droite ; même le terminal déjà rempli se retraduit.
- **Modale de fin** à la dernière étape, avec recommencer le lab ou quitter.

## Commandes simulées

**Sur Kali** : `ip addr`, `ping -c N <adresse>`, `nmap --version`, `nmap [-n] [-p <ports> | -p- | --top-ports N] [-sS | -sT | -sA | -sU | -sV | -sn] [-oN fichier] <cible>`, `cat <fichier>`, `clear`. Portée : `192.168.86.0/24` et `127.0.0.1` uniquement.

**Sur la cible** : `ip addr`, `ss -tln` / `ss -uln`, `sudo ufw status [verbose]` / `enable` / `disable` / `allow|deny|reject <port>[/tcp|udp]` / `delete ...`, `sudo systemctl stop|start|restart|status apache2|ssh`, `sudo apt update` / `install -y apache2` (carte NAT requise), `sudo tail -n N /var/log/ufw.log|/var/log/apache2/access.log`, `clear`.

`help` liste les commandes disponibles sur la machine active.

## Structure technique

Fichier HTML unique, sans dépendance externe :
- **`ENGINE`** : moteur de simulation pur (état de la cible, commandes Kali/cible, textes `Simulation : ...` traduits via `ENGINE.setLang`/`ENGINE.msg`).
- **`UI`** : dictionnaire de libellés d'interface FR/EN.
- **`MISSIONS`** : les 21 étapes (titre, sous-titre, objectif, prérequis, leçon, commande attendue, analyse), chacune taguée `part: "p1"` ou `"p2"`.
- Script d'interface : rendu des étapes, gestion du terminal, sauvegarde/reprise, bascule de langue.

## Limites

Simulation éducative : aucune commande n'agit sur un vrai réseau. Le vocabulaire et les sorties reproduisent un comportement réel mais la liste de commandes/options prises en charge est volontairement restreinte à ce que le lab enseigne.
