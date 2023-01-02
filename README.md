# Projet Final - Cours GNU/Linux B2 Ynov Aix
Benjamin Borello

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
1. Resultats


## Creations VMs
Avant de commencer à configurer les machines virtuelles nous devons créer un réseau virtuel local qui representera le reseau _Back_ : **VMNet2**
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/01%20config%20vm%20proxy/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20203751.png?raw=true)

### Serveur HAProxy
Pour le serveur HAProxy nous créons une vm Debian 11 classique. Nous devons toutefois lui ajouter une seconde carte réseau.  
La première est configurée en _bridge_ et la seconde sur le reseau virtuel précedement créé _Host-only : (VMNet2)_.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/01%20config%20vm%20proxy/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20204310.png?raw=true)

### Serveurs Web
Pour les serveurs web nous nous contenterons de n'en configurer qu'un seul. Une fois la configuration terminé il nous suffira de le cloner puis de modifier l'ip statique du clone.  
Les VMs sont egalement sur Debian 11. Cependant elles n'ont qu'une seule carte réseau configurer sur  _Host-only : (VMNet2)_.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/03%20config%20vm%20server/Capture%20d%E2%80%99%C3%A9cran%202023-01-01%20215450.png?raw=true)

## Configuration serveur HAProxy
### Carte réseau
Nous débutons par la configuration des cartes réseau du serveur.
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20041621.png?raw=true)
L'ip de la route par défaut correspond a la passerelle par défaut de l'ordinateur hébergeant les VMs. Cela permet l'accés à internet sur les différentes machines vituelles. 
Les serveurs DNS sont en automatique.

### Firewall / Ports
Nous devons désormais ouvrir le port 80 afin de pouvoir à l'avenir accéder au site web heberger sur les serveurs :
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
* Installe le services de gestion simplifié de firewall : _Uncomplicated FireWall_.
* Active UFW.
* Ouvre le port 80 utilisant le protocole TCP. Le port  80 correspond au port standard du protocole HTTP. TCP est le protocole le plus utilise pour ce type de communication. 
* Verifie que UFW est correctement executé et que les nouvelles règles sont bien appliqués.

Nous pourrions également ouvrir le port 443 à la place du 80 pour n'autoriser que les connexions sécuriser utilisant le protocole HTTPS.

### HAProxy
Nous pouvons désormais installer HAProxy :
``` shell
$ sudo apt install haproxy
[...]
```
Puis nous le configurons :
``` shell
$ sudo vi /etc/haproxy/haproxy.cfg 
```

Nous rajoutons les lignes suivantes à la fin du fichier de configuration :
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
_roundrobin_ : à chaque connexion, le serveur proposé alternera.  
1 -> 2 -> 1 -> 2 -> ...  
Il éxiste d'autres algorithmes plus efficace mais nous choisisons celui-ci pour faciliter les tests.

``` shell
$ sudo systemctl restart haproxy
```
Nous pouvons enfin redémarrer HAProxy afin d'appliquer les changements que nous venons d'effectuer.

```shell
$ sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```
Cette commande peut nous permettre de vérifier que nous n'avons pas commis d'erreurs lors de la configuration.

## Configuration serveur Apache2
### Carte reseau
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20045346.png?raw=true)  
La route par defaut correspond à l'ip de la carte reseau _Back_ du serveur HAProxy.

### Firewall / Ports
La configuration du firewall a l'aide d'ufw est strictement la meme que celle du serveur HAProxy.

### Apache2
Pour commencer nous devons installer Apache2 :
``` shell
$ sudo apt install apache2
```

Nous créons ensuite un hôte virtuel :
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

Désormais nous devons activer le nouvel hôte virtuel et désactiver celui par defaut proposé par Apache :
``` shell
$ sudo a2ensite nom_de_domaine.conf
$ sudo a2dissite 000-default.conf
```

Pour finir nous redemarrons Apache afin d'appliquer les nouveaux changements :
``` shell
$ sudo systemctl restart apache2
```



---
## Resultats
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20051147.png?raw=true)  
![](https://github.com/Nimajjj/Doc-ProjetFinal-GNU-Linux-B2/blob/main/Capture%20d%E2%80%99%C3%A9cran%202023-01-02%20051158.png?raw=true)  