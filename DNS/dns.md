# Installation du DNS unbound 



Pour notre DNS local nous allons utiliser le protocol suivant https://github.com/MatthewVance/unbound-docker 


# Unbound

Quelques mots sur la fonctionnalité unbound , unbound est un résolveur DNS récursif, qui possède une mise en cache. Il est conçu pour être rapide et léger, avec des fonctionnalités modernes basées sur des standards .

La minimisation des noms de requêtes consiste à limiter la quantité d'informations divulguées lors des requêtes DNS. Au lieu de transmettre le nom de domaine complet, seules les parties pertinentes sont envoyées, réduisant ainsi la propagation d'informations sensibles.

L'utilisation agressive des caches validés par DNSSEC optimise l'utilisation des données déjà validées par DNSSEC. Cela réduit la charge sur les serveurs DNS, améliore les performances globales et garantit l'intégrité des données grâce à la vérification des signatures numériques.

La prise en charge des zones d'autorité permet de charger une copie de la zone racine, qui est la base du système DNS. Cela permet une résolution plus rapide des requêtes, réduit la dépendance envers les serveurs de noms externes et renforce la résilience du DNS.

En combinant ces normes et techniques, on améliore la confidentialité des utilisateurs en limitant la divulgation d'informations personnelles, ainsi que la robustesse et les performances globales du DNS. Ces avancées contribuent à renforcer la sécurité et la fiabilité du système DNS dans son ensemble.


# Mis en place du DNS
Comme pour tous nos services, le serveur DNS sera hébergé dans un conteneur qui, à son tour, sera exécuté au sein d'une machine virtuelle Fedora.

Avant de lancer le docker-compose ,j'ai décider de d'abord explorer les différente façon de le mettre en place .

La premiere etais seulement de lancer l'image à l'aide de docker run . c'est l'usage standars aucun paramètre n'est pris en compte .
```
docker run \
--name=my-unbound \
--detach=true \
--publish=53:53/tcp \
--publish=53:53/udp \
--restart=unless-stopped \
mvance/unbound:latest

```

Puis j'ai explorer l'option de forwarders . Dans le cas spécifique mentionné, les forwarders sont configurés pour utiliser les serveurs DNS de Cloudflare. Cela signifie que lorsque le serveur DNS ne peut pas résoudre une requête, il transmettra cette requête aux serveurs DNS de Cloudflare pour obtenir les informations de résolution nécessaires , or il est possible de le configurer pour nous on redirigera les requetes vers le DNS de la salle qui est 10.255.255.200 . Voici à quoi ressemble notre ficher forwards-records.conf


```
forward-zone:
  # Forward all queries (except those in cache and local zone) to
  # upstream recursive servers
  name: "."

  # my DNS
  forward-addr: 10.255.255.200
```

Pour le lancer avec le run il faudra ajouter la ligne suivante 

```
---volume $(pwd)/forward-records.conf:/opt/unbound/etc/unbound/forward-records.conf:ro \
mvance/unbound:latest
```


Il est aussi possible de configurer le fichier unbound.conf . Dans notre cas la configuration de base nous suffit. 

Parlons maintenant de l'option a-records.conf . Bien que Unbound ne soit pas un serveur de noms autoritaire complet, il prend en charge la résolution des entrées personnalisées sur un petit réseau local privé. En d'autres termes, on peut utiliser Unbound pour résoudre des noms fictifs tels que your-computer.local au sein de notre LAN. 

En configurant Unbound avec des entrées de zone locale spécifiques, vous pouvez créer un environnement de résolution DNS local où vous pouvez attribuer des noms personnalisés aux appareils ou services au sein de votre LAN. Unbound sera alors capable de résoudre ces noms vers les adresses IP correspondantes ou d'autres enregistrements configurés pour ces entrées. 

 Cette fonctionnalité peut être particulièrement utile dans de petits réseaux domestiques ou de bureau où vous souhaitez utiliser des noms personnalisés pour les ressources locales sans dépendre de serveurs DNS externes.

 Voici à quoi ressemble notre fichier a-records.conf 

 ```
# A Record
local-data: "whiteness.g1.com. A 192.168.16.1"
local-data: "prox1.g1.com. A 192.168.16.2"
local-data: "prox2.g1.com. A 192.168.16.3"
local-data: "ad.g1.com. A 192.168.16.10"
local-data: "pbs.g1.com. A 10.202.16.4"
local-data: "portainer.g1.com. A 192.168.16.40"
local-data: "gitlab.g1.com. A 192.168.16.44"
local-data: "nextcloud.g1.com. A 192.168.16.45"
local-data: "ftb.g1.com. A 192.168.16.47"
local-data: "snipeit.g1.com. A 192.168.16.48"
local-data: "glpi.g1.com. A 192.168.16.50"
local-data: "wordpress.g1.com. A 192.168.16.60"
local-data: "harbor.g1.com. A 192.168.16.122"
local-data: "moodle.g1.com. A 192.168.16.126"
local-data: "scodoc.g1.com. A 192.168.16.141"

# PTR Record
local-data-ptr: "192.168.16.1 whiteness.g1.com."
local-data-ptr: "192.168.16.2 prox1.g1.com."
local-data-ptr: "192.168.16.3 prox2.g1.com."
local-data-ptr: "192.168.16.10 ad.g1.com."
local-data-ptr: "10.202.16.4 pbs.g1.com."
local-data-ptr: "192.168.16.40 portainer.g1.com."
local-data-ptr: "192.168.16.44 gitlab.g1.com."
local-data-ptr: "192.168.16.45 nextcloud.g1.com."
local-data-ptr: "192.168.16.47 ftb.g1.com."
local-data-ptr: "192.168.16.48 snipeit.g1.com."
local-data-ptr: "192.168.16.50 glpi.g1.com."
local-data-ptr: "192.168.16.60 wordpress.g1.com."
local-data-ptr: "192.168.16.122 harbor.g1.com."
local-data-ptr: "192.168.16.126 moodle.g1.com."
local-data-ptr: "192.168.16.141 scodoc.g1.com."
```


Une fois toutes ces options testé , on ce penche vers le docker-compose.yml , voici le fichier de base .
```
version: '3'
services:
  unbound:
    container_name: unbound
    image: "mvance/unbound:latest"
    expose:
      - "53"
    networks:
     - dns
    network_mode: bridge
    ports:
      - target: 53
        published: 53
        protocol: tcp
        mode: host
      - target: 53
        published: 53
        protocol: udp
        mode: host
    volumes:
      - type: bind
        read_only: true
        source: ./my_conf/forward-records.conf
        target: /opt/unbound/etc/unbound/forward-records.conf
      - type: bind
        read_only: true
        source: ./my_conf/a-records.conf
        target: /opt/unbound/etc/unbound/a-records.conf
    restart: unless-stopped

networks:
  dns:

volumes:
  mydata:
  ```



Voici aprés l'avoir adapté sur notre configuration 
```
version: '3'
services:
  unbound:
    container_name: unbound
    image: "mvance/unbound:latest"
    expose:
      - "53"
    mac_address: 02:42:c0:a8:84:41
    networks:
      x_vlan:
        ipv4_address: 192.168.16.41
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    volumes:
      - type: bind
        read_only: true
        source: ./forward-records.conf
        target: /opt/unbound/etc/unbound/forward-records.conf
      - type: bind
        read_only: true
        source: ./a-records.conf
        target: /opt/unbound/etc/unbound/a-records.conf
    restart: unless-stopped
networks:
  x_vlan:
    driver: macvlan
    driver_opts:
      parent: ens18
    ipam:
      driver: default
      config:
        - subnet: 192.168.16.0/24
          gateway: 192.168.16.99
```

une fois les fichiers mis en place sur notre VM , il suffit de lancer le docker compose pour que notre DNS tourne sur le port 53 .

<img src="Capture du 2023-06-14 09-06-18.png">

