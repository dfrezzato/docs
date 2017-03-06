# Jupyter Notebook et Rstudio Server sur AWS

## liens utiles

Sources combinées pour faire cette documentation :

- [H20 et rstudio server](http://amunategui.github.io/h2o-on-aws/)
- [Jupyter Notebook](http://chrisalbon.com/jupyter/run_project_jupyter_on_amazon_ec2.html)
- [rstudio server](https://www.rstudio.com/products/rstudio/download-server/)
- [ssh tunneling putty](https://www.digitalocean.com/community/tutorials/how-to-set-up-jupyter-notebook-for-python-3)
- [source anaconda](https://www.continuum.io/downloads#linux)
- [gestion utilisateurs sur ec2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html)

## Créer un compte sur AWS

[Amazon Web Services](https://aws.amazon.com/fr/)

Suivre les instructions et s'entraîner dans un premier temps sur les instances gratuites.

## Créer un VPC (Virtual Private Cloud)

Chercher l'outil VPC dans la liste des services :

![01](images/01-console.png "console")

Start VPC wizard :

![02](images/02-start-vpc.png)

Select :

![03](images/03-vpc-select.png)

Laisser les paramètres par défaut et indiquer le nom du VPC (ici datascience) :

![04](images/04-vpc-config.png)

Create VPC :

![05](images/05-vpc-config.png)

## Création de la VM

Choisir EC2 dans la liste des services :

![06](images/06-console.png)

Launch instance :

![07](images/07-launch-instance.png)

Choisir ubuntu server 16.04 (les instructions de configuration diffèreront si la distribution n'est pas une ubuntu) :

![08](images/08-OS.png)

Choisir le type d'instance (c'est là que se décide la puissance de calcul nécessaire) :

![09](images/09-instance-type.png)

Configure instance detail :

![10](images/10-instance-detail.png)

## Paramétrage de la VM

indiquer le nom du VPC dans Network ainsi que Enable pour Auto-assign Public IP :

![11](images/11-vpc-ip.png)

Clic sur add storage :

![12](images/12-add-storage.png)

Taille du stockage puis clic sur review and launch :

![13](images/13-storage-size-review-launch.png)

Edit security group (permet d'ouvrir les ports nécessaires pour communiquer avec l'instance ec2) :

![14](images/14-edit-security-group.png)

Ajouter les règles tcp telles qu'affichée (ports 8787 et 8888, anywhere) :

![15](images/15-tcp-rules.png)

Review and Launch :

![16](images/16-review-launch.png)

## Authentification

Clic sur launch :

![17](images/17-launch.png)

L'instance nous fournit la clé d'authentification (indiquer le ```key pair name``` puis ```download keypair```), qui servira à se connecter ultérieurement (une sorte de mot de passe qu'on télécharge au préalable sur la machine locale) :

![18](images/18-download-keypair.png)

Lancement en cours :

![19](images/19-launch-status.png)

L'instance ec2 est lancée, elle est accessible depuis son ```Public DNS``` de la forme ```ec2.xxxxxx.amazonaws.com``` :

![20](images/20-public-dns-ip.png)

Il faut maintenant s'y connecter puis installer les programmes nécessaires.

## Connexion à l'instance ec2


### Cas 1 : Machine locale sous windows

[doc aws sur putty](http://docs.aws.amazon.com/fr_fr/AWSEC2/latest/UserGuide/putty.html)

Putty est un programme permettant de se connecter à l'instance ec2 en lançant un terminal.

Putty utilise la clé d'authentification qui a été téléchargée (fichier téléchargé juste avant le lancement de la VM)

Indiquer l'adresse de l'instance (disponible depuis la console) sous forme de ubuntu@XXX :

![p01](images/putty_manuel/01-session.png)

Lorsqu'il faut passer par un proxy, indiquer les paramètres de celui-ci :

![p02](images/putty_manuel/02-proxy.png)

Saisir les paramètres qui permettent d'associer le serveur localhost:8888 de la vmlinux au lien localhost:8888 de la machine locale :

![p03](images/putty_manuel/03-ssh-tunnel.png)

Putty utilise un format de clé (fichier ```*.ppk```) qui n'est pas le même que celui qu'on a téléchargé (```*.pem```), il faut donc utiliser puttygen pour la conversion :

cliquer sur *load* et chercher l'emplacement du fichier ```*.pem``` :

![p04](images/putty_manuel/04-puttygen.png)

La clé a été convertie en fichier ```*.ppk```, cliquer sur *save private key* et choisir l'emplacement du fichier ```*.ppk``` :
![p05](images/putty_manuel/05-key-conversion.png)

On peut fermer l'application putty gen et renseigner la clé dans putty :

![p06](images/putty_manuel/06-cle-session.png)

Finalement, cliquer sur *Open* dans putty : ceci lance la connection à l'instance ec2 ainsi que le tunnel ssh (qui servira par la suite pour jupyter notebook)

### Cas 2 : Machine locale sous linux
Si notre machine locale est sous linux :

```bash
# le dossier où on a mis la keypair.pem
cd ~/Dropbox/keypair/ # endroit où on a déposé sa clé
chmod 400 datascience.pem
ssh -i "datascience.pem" ubuntu@ec2-XXXXXXX.eu-central-1.compute.amazonaws.com
```

## Installation sur la VM de Rstudio server et H2O

On est maintenant logué dans la machine virtuelle via ssh : les commandes suivantes sont saisies via ssh et exécutées par l'instance ec2.

```bash
# installation de R et rstudio server
sudo apt-get install r-base
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/rstudio-server-1.0.136-amd64.deb
sudo gdebi rstudio-server-1.0.136-amd64.deb

# h2o nécessite java et curl
sudo apt-get install libcurl4-openssl-dev
sudo apt-get install default-jre

# add user : adapter à son usage
sudo useradd david
echo "david:monpassword" | sudo chpasswd
sudo mkhomedir_helper david
```
## installation H2O

Aller sur l'url de l'instance ec2 (à récupérer dans la console AWS) qui a cette forme : http://ec2-XXXXX.eu-central-1.compute.amazonaws.com:8787

S'identifier avec son user (dans l'exemple : david) et son mot de passe (dans l'exemple : monpassword) puis lancer les commandes suivantes dans R studio server :

```R
install.packages("h2o")
library(h2o)
localH2O = h2o.init()
demo(h2o.glm)
demo(h2o.gbm)
```


## installation python

Après s'être connecté à l'instance ec2 via ssh (cf section précédente) :

```bash
cd ~
wget https://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh
bash Anaconda3-4.3.0-Linux-x86_64.sh
source ~/.bashrc

mkdir ~/Notebooks
tmux # on veut lancer le serveur dans un onglet tmux
cd ~/Notebooks
jupyter notebook
# bien récuperer l'url indiquée lors du lancement de la commande.
# si on oublie, on peut lancer la commande : jupyter notebook list
# l'url a la forme : http://localhost:8888/?token=6cf7df71be234f35e86XXXXc326e150390fadfcdcbe5e94a
```

Bien penser à récupérer l'url du notebook.

*controle-b d* permet de quitter tmux et le serveur (lancé par la commande jupyter notebook) continuera de tourner.

## Jupyter notebook via SSH tunneling

Permet d'associer localhost:8888 sur le pc local au localhost:8888 (sur lequel écoute le serveur jupyter notebook) de l'instance ec2 via ssh.

Si on a un pc local sous linux, on utilise cette commande :

```bash
# sous la machine locale, commande linux
cd ~/Dropbox/keypair/ # endroit où on a déposé sa clé
ssh -i "datascience.pem" -L 8888:localhost:8888 ubuntu@ec2-XXXXXXX.eu-central-1.compute.amazonaws.com
```
Si le pc local est sous windows et que putty a été paramétré dans les sections précédentes, le tunnel SSH est déjà en place.

Aller à l'url http://localhost:8888/?token=XXX depuis son navigateur.

## Jupyter Notebook avec mot de passe
