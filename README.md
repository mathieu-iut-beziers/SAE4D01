# TP/TD sécurité Docker

## 1 Pré-requis, recommandations et notation du TP

### 1.1 Installation de Docker et obtenir de l’aide

#### 1.1.1 Rappel: Installation de Docker sous Linux

#### 1.1.2 Aide sur Docker

## 2 Peurs sur les containers

### 2.1 Etat des lieux rapide de la sécurité des containers

### 2.2 Les menaces

## 3 Utilisation des namespaces par Docker

### 3.1 Accéder au namespace de l’hôte depuis un container Docker c’est mal

Je lance le container avec sudo docker run -it --rm --pid=host --net=host ubuntu, et en realisant un htop sur l'hote et un top sur le container et ont voit bien les deux processuce dans la vm et dans le container.

```bash
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
17902 1000      20   0    8488   4684   3520 S   0.7   0.1   0:00.22 htoppow
17903 root      20   0    7592   3796   2948 R   0.7   0.1   0:00.08 top
```

### 3.2 Utilisation des usernamespaces par Docker afin de limiter les droits d’un attaquant

> `systemctl restart docker` pour restart

Edition de /etc/docker/daemon.json :

```bash
{
  "userns-remap": "dovi"
}
```

Pour /etc/subuid et /etc/subgid : dovi:231072:65536

Apres avoir fait ça, les procesus sont bien maper entre les container et la vm.

### 3.3 Contrôle des ressources allouées aux processus d’un container

#### 3.3.1 Contrôle des ressources des containers au travers des cgroups

Par default quand on le lance on a une consomation de 100% des deux cpu de la VM :

![img](img/Capture%20d’écran%20du%202023-06-05%2021-18-57.png)

Qand on le limite a un seul cpu on le constate bien sur le htop :

![img](./img/Capture%20d’écran%20du%202023-06-05%2021-20-16.png)

#### 3.3.2  Lutte contre l’épuisement des ressources du container et de l’hôte par déni de servicelocal ( "fork bomb" par exemple ).

> `docker stats --no-stream=True` -> permet lister les resources de container
> `dockerexec-it deb1 /bin/bash -c":(){ :|:& };:"` -> lance un container avec une fork bomb

L'option `--pids-limit 100` permet de limiter le nombre de procesus et donc limiter le risque qu'un fork bomb paralise la machine principale.

## 4  Sécurisation des capacités données à un container Docker

### 4.1  Création d’un container privilégié

> `apt update -y && apt install libcap2-bin libcap-ng-utils -y` -> packet necesaire
> `capsh --print` -> commande pour afficher les capabilities
> `pscap -a` -> commande pour afficher les capabilities du bash

Retour d'un container non priviligier :

```bash
root@196ca4d48109:/# capsh --print
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Ambient set =
Current IAB: !cap_dac_read_search,!cap_linux_immutable,!cap_net_broadcast,!cap_net_admin,!cap_ipc_lock,!cap_ipc_owner,!cap_sys_module,!cap_sys_rawio,!cap_sys_ptrace,!cap_sys_pacct,!cap_sys_admin,!cap_sys_boot,!cap_sys_nice,!cap_sys_resource,!cap_sys_time,!cap_sys_tty_config,!cap_lease,!cap_audit_control,!cap_mac_override,!cap_mac_admin,!cap_syslog,!cap_wake_alarm,!cap_block_suspend,!cap_audit_read,!cap_perfmon,!cap_bpf,!cap_checkpoint_restore
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=0(root) euid=0(root)
gid=0(root)
groups=0(root)
Guessed mode: UNCERTAIN (0)
root@196ca4d48109:/# pscap -a 
ppid  pid   name        command           capabilities 
0     1     root        bash              chown, dac_override, fowner, fsetid, kill, setgid, setuid, setpcap, net_bind_service, net_raw, sys_chroot, mknod, audit_write, setfcap
```

Retour d'un container priviligier :

```bash
root@6aff9d4456bb:/# capsh --print
Current: =ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore 
Ambient set = 
Current IAB:
Securebits: 00/0x0/1'b0 
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=0(root) euid=0(root)
gid=0(root)
groups=0(root)
Guessed mode: UNCERTAIN (0)
root@6aff9d4456bb:/# pscap -a 
ppid  pid   name        command           capabilities 
0     1     root        bash              full
```

> `--privileged` donne accer a :

- cap_chown            -> Modification propriétaires fichiers
- cap_dac_override     -> Contournement contrôles accès aux fichiers
- cap_fowner           -> Modification propriétaire fichiers
- cap_fsetid           -> Modification utilisateur et groupe d'un fichier
- cap_kill             -> Envoi signaux à d'autres processus
- cap_setgid           -> Modification identifiant groupe
- cap_setuid           -> Modification identifiant utilisateur
- cap_setpcap          -> Modification capacités processus
- cap_net_bind_service -> Liaison ports réseau inférieurs 1024
- cap_net_raw          -> Accès brut sockets réseau
- cap_sys_chroot       -> Utilisation chroot
- cap_mknod            -> Création fichiers spéciaux
- cap_audit_write      -> Écriture journal système
- cap_setfcap          -> Modification capacités processus

`--privileged` donne accès à toutes les capacités du système.

### 4.2  Prise de contrôle du container avec des capabilities permettant une es-calade de privilèges

Dans l'image stress je rajoute les service ssh necesaire a la manipulation.

```bash
docker run -it --cap-drop ALL --cap-add CAP_NET_BIND_SERVICE -p 2222:22 jmp/stress
```

Pour se connecer en ssh : `ssh -p 2222 root@localhost`

## 5  Attaque sur le daemon Docker par un utilisateur local à l’hôte

> `docker run -it -v /etc/passwd:/host-etc-passwd -v /usr/sbin:/host-usr-sbin ubuntu` -> permet de recuperer les ficher demandé

Pouvoir fair ça pose de gros probleme, cela peut permetre d'exposer pbliquement des donnée sensible.

## 6  Utilisation de AppArmor afin de contrôler un container vul-nérable à ShellShock

On peut donc bien executer le shell dans le container avec un docker exec.

Apres avoir fait la conf on l'exporte puis on peut l'inclur dans le container avec la commande de lancement : `docker run --security-opt="apparmor:shellshock"`

## 7  Utilisation d’un scanneur de vulnérabilités

Pour scanée les vulnerabilitée on peut utiliser InSpec : `https://www.prplbx.com/resources/blog/docker-part2/`.

## 8  Utilisation de SecComp pour limiter l’utilisation de certainsappels systèmes

Ficher de conf :

```json
{"defaultAction":"SCMP_ACT_ALLOW","syscalls":[{"name":"chmod","action":"SCMP_ACT_ERRNO"}]}
```

Commande de lancement :

```bash
docker run --rm \
             -it \
             --security-opt seccomp=/home/dovi/tp/chmod.json \
             ubuntu
```

On peut donc plus realiser de chmod dans le container.

## 9  Containers en lecture seule

```dockerfile
FROM nginx:latest
VOLUME /tmp
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Pour lancer le container `docker run -d -p 80:80 -v /tmp:/tmp container-web`

## 10  Contrôle de l’élévation de privilèges dans un container vial’option "-security-opt=no-new-privileges"

L'option "--security-opt=no-new-privileges" utilisée lors de l'exécution du conteneur Docker a un effet sur les privilèges de l'utilisateur à l'intérieur du conteneur.

## 11  Rootless containers

## 12  Annexe : Débugguer un container