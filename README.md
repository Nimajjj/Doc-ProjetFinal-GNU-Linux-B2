# Projet Final - Cours GNU/Linux B2 Ynov Aix
Benjamin Borello - Ynov Aix 2022/2023

## Sommaire
1. Creation VMs
    * Serveur HAProxy
    * Serveurs WEB
1. Configuration serveur HAProxy
    * Carte reseau
    * Firewall / Ports
    * HAProxy
1. Configuration serveur WEB
    * Carte reseau
    * Firewall / Ports
    * Apache2


## Creations VMs
Avant de commencer a configurer les machines virtuelles nous devont creer un reseau virtuel local qui representera le reseau "Back" : **VMNet2**
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/01%20config%20vm%20proxy/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20203751.png?raw=true)

### Serveur HAProxy
Pour le serveur HAProxy nous creons une vm Debian 11 classique. Nous devons toutefois lui ajouter une seconde carte reseau.
La premiere est configure en _bridge_ et la seconde sur le reseau virtuel precedement cree _Host-only : (VMNet2)_.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/01%20config%20vm%20proxy/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20204310.png?raw=true)

### Serveurs Web
Pour les serveurs web nous nous contenterons de n'en creer et configurer qu'un seul. Une fois la configuration termine il nous suffira de le cloner puis de modifier l'ip statique du clone.
Les VMs sont egalement sur Debian 11. Cependant elles n'ont qu'une seule carte reseau configurer sur  _Host-only : (VMNet2)_.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/03%20config%20vm%20server/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20215450.png?raw=true)

## Configuration serveur HAProxy
### Carte reseau
Nous debutons par la configuration des cartes reseau du serveur.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20041621.png?raw=true)
L'ip de la route par defaut correspond a la passerelle par defaut de l'ordinateur hebergeant les VMs. Cela permet l'acces a internet sur les differents machines vituelles. 
Les serveurs DNS sont en automatique.

### Firewall / Ports
Nous devons desormais ouvrir le port 80 afin de pouvoir a l'avenir acceder au site web heberger sur les serveurs :
``` shell
$ sudo apt install ufw
[...]
$ sudo ufw enable
$ sudo ufw allow 80/tcp
Rules updated
Rules updated (v6)
$ sudo ufw status
Status: active

To              Action      From
--              ------      ----
80/tcp          ALLOW       Anywhere
80/tcp (v6)     ALLOW       Anywhere (v6)
```
Dans l'ordre des commandes :
* Installe le services de gestion simplifie de firewall : _Uncomplicated FireWall_.
* Active UFW.
* Ouvre le port 80 utilisant le protocole TCP. Le port  80 correspond au port standard du protocole HTTP. TCP est le protocole le plus utilise pour ce type de communication. 
* Verifie que UFW est correctement execute et que les nouvelles regles sont bien appliques.

Nous pourrions egalement ouvrir le port 443 a la place du 80 pour n'autoriser que les connexions securiser utilisant le protocole HTTPS.

### HAProxy
Nous pouvons desormais installer HAProxy :
``` shell
$ sudo apt install haproxy
[...]
```
Puis nous le configurons :
``` shell
$ sudo vi /etc/haproxy/haproxy.cfg 
```

Nous rajoutons les lignes suivantes a la fin du fichier de configuration :
```
# FRONTEND
frontend apache_front
        # Port d'ecoute du serveur "Front"
        bind *:80
        # Definit le protocole utilise
        mode tcp
        # Definit le back end par defaut
        default_backend    apache_backend_servers
  
# BACKEND
backend apache_backend_servers
        # Definit l'algorithme d'equilibrage du trafique
        balance            roundrobin
        # Link les adresses des deux serveurs web
        server             backend01 192.168.2.101:80 check
        server             backend02 192.168.2.102:80 check

```
_roundrobin_ a chaque connection le serveur propose alternera.
Il existe d'autre algorithme plus efficace (qui peuvent rediriger sur le serveur avec le moins de connection par exemple) mais nous choisisons celui-ci pour faciliter les tests.

``` shell
$ sudo systemctl restart haproxy
```
Nous pouvons enfin redemarrer HAProxy afin d'appliquer les changement que nous venons d'effectuer.

```shell
$ sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```
Cette commande peut nous permettre de verifier que nous n'avons pas commis d'erreur lors de la configuration.

## Configuration serveur Apache2
### Carte reseau
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20045346.png?raw=true)  
La route par defaut correspond a l'ip de la carte reseau _Back_ du serveur HAProxy.

### Firewall / Ports
La configuration du firewall a l'aide d'ufw est strictement la meme a celle du serveur HAProxy.

### Apache2
Pour commencer nous devons installer Apache2 :
``` shell
$ sudo apt install apache2
```

Nous creons ensuite un hote virtuel :
``` shell
$ sudo mkdir -p /var/www/nom_de_domaine
$ sudo vi /var/www/nom_de_domaine/index.html
$ sudo vi /etc/apache2/sites-available/nom_de_domaine.conf
```
Dans l'ordre des commandes :
* Creation du repertoire qui contiendra tous les fichiers du site web
* Creation d'un fichier html basique
* Creation du fichier de configuration de l'hote virtuel

> /etc/apache2/sites-available/nom_de_domaine.conf :
``` 
<VirtualHost *:80>
    ServerAdmin admin@nom_de_domaine
    ServerName nom_de_domaine
    ServerAlias www.nom_de_domaine
    DocumentRoot /var/www/nom_de_domaine
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Desormais nous devons activer le nouvel hot virtuel et desactiver celui par defaut proposer par Apache :
``` shell
$ sudo a2ensite nom_de_domaine.conf
$ sudo a2dissite 000-default.conf
```

Pour finir nous redemarrons Apache afin d'appliquer les nouveaux changements :
``` shell
$ sudo systemctl restart apache2
```