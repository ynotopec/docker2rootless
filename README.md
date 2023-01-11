# docker2rootless
```bash
####### BEGIN USEFULL CODE ########
image=

export REGISTRY_FQDN_DST=
####### END USEFULL CODE ########

apt install podman podman-docker -y ||yum install podman podman-docker -y
systemctl start podman.socket
cat <<EOT >Dockerfile
FROM ${image}
USER root
####### BEGIN USEFULL CODE ########

####### END USEFULL CODE ########
RUN grep -rlw http /etc |while read i ;do sed -ri 's#\b80\b#8080#' \${i} ; sed -ri 's#\b443\b#6443#' \${i} ;done
RUN find / -xdev -exec chmod -f -c g=u {} \; -exec chown -f -c 1001:root {} \; >/dev/null 2>&1
RUN grep -w 1001 /etc/passwd ||echo "rootless:x:1001:0:rootless:/root:/bin/bash" >>/etc/passwd
USER 1001
EOT
podman rmi -f ${image}
podman build -t ${image} . |tee /tmp/${image}.build
podman login ${REGISTRY_FQDN_DST} &&\
podman push ${image} ${REGISTRY_FQDN_DST}/${image}
```
OPENAI comment

Ce code semble être un script pour construire et pousser une image Docker. Le script commence par installer les paquets "podman" et "podman-docker" en utilisant apt ou yum, selon le système sur lequel il est en cours d'exécution. Il démarre ensuite le service "podman.socket".

Ensuite, il crée un fichier Dockerfile dans le répertoire courant, en utilisant la variable image comme image de base. Il existe une section de commentaires étiquetée "####### BEGIN USEFULL CODE ########" et "####### END USEFULL CODE ########", qui est vide dans ce script. Cela suggère qu'il faut ajouter des instructions supplémentaires entre ces commentaires.

Après la section de commentaires, le script exécute plusieurs commandes :

    grep -rlw http /etc |while read i ;do sed -ri 's#\b80\b#8080#' ${i} ; sed -ri 's#\b443\b#6443#' ${i} ;done : Cette commande utilise grep et sed pour rechercher tous les fichiers dans le répertoire /etc récursivement pour la chaîne "http" et remplace toutes les occurrences de "80" avec "8080" et toutes les occurrences de "443" avec "6443".
    find / -xdev -exec chmod -f -c g=u {} \; -exec chown -f -c 1001:root {} \; >/dev/null 2>&1 : Cette commande utilise la commande find pour rechercher tous les fichiers et répertoires à partir de / récursivement et définir le propriétaire sur 1001 et le groupe sur root et assure qu'ils ont les mêmes permissions que leur répertoire parent.
    grep -w 1001 /etc/passwd ||echo "rootless:x:1001:0:rootless:/root:/bin/bash" >>/etc/passwd : Cette commande va vérifier si l'utilisateur 1001 est présent dans /etc/passwd et s'il n'est pas présent elle va ajouter un nouvel utilisateur rootless avec l'ID d'utilisateur 1001.

Ensuite, le script bascule vers l'utilisateur 1001 avec USER 1001 et termine le fichier Dockerfile. Après cela, il va supprimer l'ancienne image avec podman rmi -f ${image}

Enfin, il utilise l'outil en ligne de commande Podman pour construire la nouvelle image à partir du fichier Dockerfile et la variable REGISTRY_FQDN_DST est utilisée pour spécifier le nom de domaine entièrement qualifié du registre de conteneurs où l'image sera poussée. La dernière commande `podman login ${REGISTRY_FQDN_DST} && podman push ${image} ${REGISTRY_FQDN_DST}/${image} se connecte au registre spécifié, et pousse l'image construite dans le registre et le dépôt spécifiés.

Il est important de noter que la construction est créée en utilisant cat << EOT>Dockerfile qui est un here-document. Il permet de créer une chaîne multiligne sans avoir à concaténer manuellement les lignes, et il se termine avec EOT à la dernière ligne. 
