# TD2-Running-a-node-to-accept-Bitcoin-payment



## Install Dependencies
Pour utiliser le BTCPayServer il faut install .NET Core SDK, NBXplorer, et une database

### .NET Core SDK (installation)
```bash
$wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
$sudo dpkg -i packages-microsoft-prod.deb
$sudo apt-get install apt-transport-https
$sudo apt-get update
$sudo apt-get install -y dotnet-sdk-3.1
```
### .NET Core SDK (check)
```bash
$dotnet --info
```

### PostgreSQL (installation)
```bash
$sudo apt install postgresql postgresql-contrib
```
### PostgreSQL (check) 
```
$psql --version
$sudo systemctl status postgresql
$sudo -u postgres psql
```

### NBXplorer (installation)
```bash
$cd ~
$git clone https://github.com/dgarage/NBXplorer
$cd NBXplorer
$git checkout latest
$./build.sh
```


## Adapt UFW Config
Permet d'ouvrir les ports, autorisé les trafics: SSH(22) ; BTCPAY(80,443) ; LND(9735) ; Tor(9050,9051) ; NBXplorer(24445) ; BTCPayServer(23001) ; ... vont être ajouter au fur et à mesure

```bash
#Installation
$sudo apt install ufw
#Mode admin
$sudo su
#Acceptation des ports
$ufw allow #port comment #LeNom
#Reboot
$ufw enable
$systemctl enable ufw
#Checker les ports
$ufw status
#Quitter
$exit
```


## Adapt SSH Config
Permet de donner accès à certaines personnes dans le VM

### Putty Configuration:
```
1) Put host name or IP
2) (Connection type) SSH
3) saved Sessions -> load -> save
4) connection -> SSH -> Auth -> put private_key was generated by Puttygen
5) save
6) Open Session
```

### On the Virtual Machine:
```bash
$ sudo nano /etc/ssh/sshd_config
#We paste our ssh-rsa pubkey in the file
#We can then Restart the SSH daemon, then exit and log in again.
$ sudo systemctl restart sshd
$ exit
```


## Install BTC Pay Server
Nous allons voir l'installation du BTCPayServer, il faut aussi avoir les installations de Bitcoind et de LND

### BTCPayServer(Installation)
```bash
$cd ~
$git clone https://github.com/btcpayserver/btcpayserver.git
$cd btcpayserver
$git checkout latest
$./build.sh
$./run.sh -h
```

### Bitcoind(Installation)
```bash
$sudo adduser bitcoin
$sudo adduser administrateur1
$sudo su - bitcoin
$nvim ~/.bitcoin/bitcoin.conf 
```
```
#[Bitcoin.conf]
daemon=1
testnet=1
server=1
txindex=1
listen=1
listenonion=1
rpcuser= #nom utilisateur
rpcpassword= #mot de passe
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28332
```
Lancement et téléchargement des blocks (risque de prendre du temps)
```bash
$bitcoind
#ou
$bitcoin --testnet
```
vérification du dernier block, addresse, balance: on peut avoir les faucets sur ce site => [tBTC](https://bitcoinfaucet.uo1.net/)
```bash
#dernier block info
$bitcoin-cli getblockchaininfo
#adresse
$bitcoin-cli getnewadress
#balance
$bitcoin-cli getbalance
```
Wallet address : [2N345zy6NnZLcNkLdt99Sa2MRhZmefTpY6z](https://live.blockcypher.com/btc-testnet/address/2N345zy6NnZLcNkLdt99Sa2MRhZmefTpY6z/)

### LND(installation)
On a pas Réussit à avoir le réseau Lightning, on a pas pu avoir le node;
il fallait installer GO puis Lightning (LND)
Ce tuto permet normalement d'installer LND : [Get_LND](https://stadicus.github.io/RaspiBolt/raspibolt_40_lnd.html)

## Configure Your BTC Pay Server
Configuration du server pour avoir les clés API et donc intéragir

### Configuration du BTCPayServeur
```bash
$Mkdir -p ~/.btcpayserveur/Main
$Cd ~/.btcpayserver/Main
$Sudo vi settings.config 
```
```
#[setting.config]
network=testnet
port=23001
bind=0.0.0.0
chains=btc
BTC.explorer.url=http://127.0.0.1:24445
BTC.lightning=type=lnd-rest;server=https://127.0.0.1:8080/;macaroonfilepath=~/.lnd/data/chain/bitcoin/testnet/admin.macaroon;certthumbprint= #LND's_certificate_fingerprint
postgres=User ID + User Pssw #Postgres_DatabaseName;Password=#Prosgrs_DatabassePssw;Host=localhost;Port=5432;Database=btcpayserver;
```
Avoir le LND Fingerprint pour #LND's_certificate_fingerprint
```bash
$openssl x509 -noout -fingerprint -sha256 -inform pem -in ~/.lnd/tls.cert
```
Creer la database PostgreSQL pour DbName et Password
```bash
$sudo -i -u postgres
$creauser --pwprompt --iteractive
name: #Username (administrateur1)
password: #pssw
:n
:y
:n
$createdb -O administrateur1 btcpayserver
$exit
```

il faut aussi configurer les services: nbxplorer.service / btcpayserver.service
(ils sont sur le pdf (TD2))

### Check Configuration 
```bash
$usr/bin/dotnet run -p ~i/source/btcpayserver/BTCPayServer/BTCPayServer.csproj -c ~/.btcpayserver/Main/settings.config --network=testnet
```

### Lauch BTCPayServer
```bash
$sudo systemctl enable btcpayserver.service
$sudo service btcpayserver start
$sudo service btcpayserver status
```


## Install Website
Création du Site avec WordPress

### PHP et Apache(installation)
```bash
$sudo apt update
$sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
                 php-bcmath \
            php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip

```

### WordPress(Installation)
```bash
$sudo mkdir -p /srv/www
$sudo chown www-data: /srv/www
$curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```

### link WordPress et PostgreSQL
```bash
$ cd wp-content
$ git clone https://github.com/kevinoid/postgresql-for-wordpress.git
$ mv postgresql-for-wordpress/pg4wp Pg4wp
$ rm -rf postgresql-for-wordpress
$ cp Pg4wp/db.php db.php
$cd /var/www/html/test-with-postgres cp -rp wp-config-sample.php wp-config.php
```
```
#[wp-config.php]
/** DbName */
define('DB_NAME', #DbName(btcpayserver));
/** Username */
define('DB_USER', #Username(administrateur1));
/** Password */
define('DB_PASSWORD', #Password);
/** Hostname */
define('DB_HOST', 'localhost'(127.0.0.1:8080));
```


L'ensemble du chemin de création de l'API se trouve dans le PDF (TD2)
Pour accéder au service BTCPayserveur, il faut se créer un compte:
Create account
[Testnet_BTCPay](https://testnet.demo.btcpayserver.org/)
my wallet address cannot match with the pattern, you can create another by this: [Get_Bitcoin_Wallet](https://iancoleman.io/bip39/)

my new wallet address after => [tb1q6654eak053mz2tl4ms6mlj8vw8zf2jxldx96hy](https://live.blockcypher.com/btc-testnet/address/tb1q6654eak053mz2tl4ms6mlj8vw8zf2jxldx96hy/)

On a bien installé WordPress pour le Front, mais [Vue.js](https://vuejs.org/) reste la meilleure option !!!
#### Installation
* prendre le dossier
* cd client
* npm i
* npm i node_module
* npm i nodemon
* npm run serve (npm start)

## Create a Button to Pay With tBTC
#### HTML
```html
</div>
    <div>
    <form method="POST" action="https://testnet.demo.btcpayserver.org/apps/3V7dQqRCh8hBSGk2BDn5GZU6HQTb/pos">
    <input type="hidden" name="amount" value="100" />
    <button class="btnBTC" type="submit">Payment en Wallet BTC</button>
  </form>
  </div>
```
#### CSS
```css
.btnBTC {
  margin-top: 10%;
  display: flex;
  align-content: center#b66912;
  justify-content: center;
  color: #fff;
  background-color: #198754;
  border-color: #198754;
  padding-bottom: 30px;
  height: 20%;
  width: 50%;
  font-size: 30px;
}
```

## Create a Button to Pay With Lightning
#### HTML
```html
<div>
    <form method="POST" action="https://testnet.demo.btcpayserver.org/apps/#API_LND">
    <input type="hidden" name="amount" value="100" />
    <button class="btnLND" type="submit">Payment via Lightning Network</button>
  </form>
  </div>
```
#### CSS
```css
.btnLND {
  margin-top: 12%;
  display: flex;
  align-content: center;
  justify-content: center;
  color: #fff;
  background-color: #b66912;
  border-color: #b66912;
  padding-bottom: 30px;
  height: 20%;
  width: 50%;
  font-size: 30px;
}
```



## Documentation
* [SSH UFW Bitcoind LND(installation)](https://stadicus.github.io/RaspiBolt)
* [BTCPayServer Light Deployement](https://freedomnode.com/blog/how-to-setup-btc-and-lightning-payment-gateway-with-btcpayserver-on-linux-manual-install/)
* [BTCPayServer Full Deployement](https://docs.btcpayserver.org/ManualDeploymentExtended/)
* [Install WordPress](https://ubuntu.com/tutorials/install-and-configure-wordpress#1-overview)
* [Link WordPress and PostegreSQL](https://medium.com/@shoaibhassan_/install-wordpress-with-postgresql-using-apache-in-5-min-a26078d496fb)











