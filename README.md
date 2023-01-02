# Projet Final - Cours GNU/Linux B2 Ynov Aix
Benjamin Borello | Ynov Aix 2022/2023

## Sommaire
1. Creation VMs
    * Serveur HAProxy
    * Serveurs WEB
1. Configuration HAProxy
1. Configuration serveur Apache2


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

## Configuration HAProxy
## Configuration serveur Apache2

