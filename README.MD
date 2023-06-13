# SAE DevCloud_d01 2023

Nous somme a l'IUT de Cette, cet IUT vient d'ouvrir et donc tout est à mettre à place. On nous demande d'implémenter tous les services possibles de l'IUT sous forme **containairisée**. Il faut mettre en place de la haute disponibilité pour certains services et si possible avec de l'équilibrage de charge.

Vous devez donc implémenter à minima les services réseaux suivants:

- Un DNS en haute disponibilité (via IPVS).
  - Un DNS  (bind9). (qui sera dans le cloud)
  - Un DNS (unbound).

- Un dhcp.
- Un serveur FTP.
- Au moins un client Windows connecté à un contrôleur de domaine Samba 3 ou 4.
- Un service glpi d'inventaire. (Vos matériels y seront référencés)

- Un firewall (pfsense)
- Une solution permettant de gérer les containers de façon centralisé (A l'exclusion d'un orchestrateur de container comme kubernetes) comme Portainer.

Et les services applicatifs containairisés suivants à destination des usagers:

- Un serveur Gitlab qui vous servira de stockage de vos configurations.
- Un serveur web IUT à base d'un "Content" Management System" comme celui de l'IUT de Béziers. HA proxy assurera l'équilibrage de charge et la haute disponibilité.
  Vous testerez en charge cette solution.
- Un serveur de note Scodoc obligatoirement containérisé.
- Un serveur Nextcloud.
- Un serveur Samba. (le client Windows l'utilisera).
- Un Kanban de votre choix internalisé ou externalisé.
  
Un stockage partagé NFS permettra de survivre à l'arrêt d'un serveur pour les containers redondés.
Le choix de la solution de virtualisation est de votre ressort ainsi que l'hébergeur Cloud pour le DNS.
Vous utiliserez obligatoirement plusieurs solutions de containairisation avec au moins un service Docker, Podman, et Lxc. Un durcissement des containers est souhaité.(rootlesss, limits..). Pour Docker l'utilisation de fichier compose est fortement recommandé.

Le Jeudi 15 Juin le matin aura lieu à une présentation de votre installation au directeur. Un bilan chiffré du travail de chacun y sera présenté.
Un compte-rendu écrit et un repos git sont obligatoires.