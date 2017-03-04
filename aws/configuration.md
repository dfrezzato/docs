# Jupyter Notebook et Rstudio Server sur AWS

## liens utiles

Sources combinées pour faire cette documentation :

- [H20 et rstudio server](http://amunategui.github.io/h2o-on-aws/)
- [Jupyter Notebook](http://chrisalbon.com/jupyter/run_project_jupyter_on_amazon_ec2.html)
- [rstudio server](https://www.rstudio.com/products/rstudio/download-server/)

## Créer un compte sur AWS

[Amazon Web Services](https://aws.amazon.com/fr/)

Alerte payant


## Créer un VPC (Virtual Private Cloud)

## Création de la VM

## Putty pour la connexion ssh

## Installation sur la VM de Rstudio server et H2O

```bash
# le dossier où on a mis la keypair.pem
cd ~/Dropbox/keypair/
chmod 400 datascience.pem
ssh -i "datascience.pem" ubuntu@ec2-XXXXXXX.eu-central-1.compute.amazonaws.com

# installation de R et rstudio server
sudo apt-get install r-base
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/rstudio-server-1.0.136-amd64.deb
sudo gdebi rstudio-server-1.0.136-amd64.deb

# h2o nécessite java et curl
sudo apt-get install libcurl4-openssl-dev
sudo apt-get install default-jre

# add user
sudo useradd david
echo "david:monpassword" | sudo chpasswd
sudo mkhomedir_helper david
```

```R
install.packages("h2o")
library(h2o)
localH2O = h2o.init()
demo(h2o.glm)
demo(h2o.gbm)
```
