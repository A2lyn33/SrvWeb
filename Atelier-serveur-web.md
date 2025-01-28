# SrvWeb
---
# 🌐 Mise en place d’un Serveur Web avec Reverse Proxy

## ✔️ **Étape 1 - Prérequis**
Pour cet atelier, assure-toi d’avoir :
- 🖥️ **Hyperviseur** : VirtualBox pour créer des machines virtuelles (VM).
- 💾 **2 VM sous Debian 12** :
  - **VM webserver** :
    - Carte réseau : Accès par pont en DHCP.
    - Configuration SSH fonctionnelle.
  - **VM proxy** :
    - Carte réseau : Accès par pont en DHCP.
    - Configuration SSH fonctionnelle.
- 📡 **Connexion Internet** via une box opérateur.
- 📱 **Appareil 4G/5G** pour tester depuis un autre réseau.
- 🌍 **Compte No-IP** pour un nom de domaine dynamique : [Créer un compte](https://my.noip.com/).

💡 **Conseil** : Ce guide est testé sur Debian 12 avec VirtualBox 7, mais des ajustements peuvent être nécessaires pour d’autres versions.

---

## 🔧 **Étape 2 - Installation d’Apache**
1. Sur la **VM webserver**, exécute les commandes suivantes pour installer Apache :
   ```bash
   apt update && apt upgrade -y
   apt install apache2 -y
   ```
2. Vérifie le statut d’Apache :
   ```bash
   systemctl status apache2
   ```
3. Récupère l’adresse IP privée de la VM via `ip addr`. Ouvre un navigateur sur ta machine hôte et accède à l’IP :
   ```http://Adresse_IP_privée```

---

## 📝 **Étape 3 - Configuration de la page d’accueil**
1. Modifie la page d’accueil située dans `/var/www/html/index.html` :
   - Remplace son contenu par :
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Welcome to My Server</title>
       <style>
           body { font-family: Arial, sans-serif; background-color: #f4f4f4; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
           .container { text-align: center; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); }
           h1 { color: #333; }
           .button { margin-top: 20px; padding: 10px 20px; color: white; background-color: #007BFF; border: none; border-radius: 5px; text-decoration: none; }
           .button:hover { background-color: #0056b3; }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>Bienvenu sur mon serveur Personnel !</h1>
           <a href="next.html" class="button">OK</a>
       </div>
   </body>
   </html>
   ```
2. Ajoute un fichier `next.html` dans le même dossier avec ce contenu :
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Choose a Site</title>
       <style>
           body { font-family: Arial, sans-serif; background-color: #f4f4f4; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
           .container { text-align: center; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); }
           h1 { color: #333; }
           .button { margin: 10px; padding: 10px 20px; color: white; background-color: #007BFF; border: none; border-radius: 5px; text-decoration: none; }
           .button:hover { background-color: #0056b3; }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>Choose a Site to Visit</h1>
           <a href="https://www.google.com" class="button">Google</a>
           <a href="https://www.wikipedia.org" class="button">Wikipedia</a>
           <a href="https://www.wildcodeschool.com" class="button">WCS</a>
           <a href="index.html" class="button">Home</a>
       </div>
   </body>
   </html>
   ```
3. Redémarre Apache et teste dans ton navigateur :
   ```bash
   systemctl restart apache2
   ```

---

## 🌍 **Étape 4 - Configuration de la box Internet**
1. Accède à l’interface d’administration de ta box (souvent via une IP comme `192.168.1.254`).
2. Ajoute une règle de redirection de port (PAT) :
   - Port source (externe) : `80`
   - Port destination (interne) : `80`
   - IP : Adresse privée de ta VM webserver.
3. Teste avec un appareil connecté à un autre réseau (4G/5G) via l’IP publique obtenue sur [mon-ip.io](https://mon-ip.io).
4. Sécurise en modifiant la redirection pour utiliser un port externe non standard (ex. `22545`).

---

## 🌐 **Étape 5 - Enregistrement d’un nom de domaine**
1. Connecte-toi sur [No-IP](https://my.noip.com).
2. Dans **Dynamic DNS → NO-IP Hostnames**, crée un nom d’hôte :
   - **Hostname** : Nom personnalisé (ex. `HomeHomeWCS`).
   - **Domain** : Sélectionne un domaine (ex. `webhop.me`).
   - **Record Type** : A
   - **IPV4 Address** : Adresse IP publique de ta box.
3. Teste avec l’URL : `http://HomeHomeWCS.webhop.me:22545`.

---

## 🛡️ **Étape 6 - Mise en place d’un Reverse Proxy**
1. Installe et configure Apache sur la VM **proxy** :
   ```bash
   apt install apache2 -y
   a2enmod proxy
   a2enmod proxy_http
   a2enmod proxy_balancer
   a2enmod lbmethod_byrequests
   systemctl restart apache2
   ```
2. Crée un nouveau fichier `/etc/apache2/sites-available/000-default.conf` :
   ```apache
   <VirtualHost *:22545>
       ServerName homehomewcs.webhop.me

       ProxyPreserveHost On
       ProxyPass / http://192.168.1.100:80/
       ProxyPassReverse / http://192.168.1.100:80/

       <Location />
           Order allow,deny
           Allow from all
       </Location>
   </VirtualHost>
   ```
3. Ajoute un port d’écoute dans `/etc/apache2/ports.conf` :
   ```plaintext
   Listen 22545
   ```
4. Active le site et redémarre Apache :
   ```bash
   a2ensite 000-default.conf
   systemctl restart apache2
   ```
5. Configure la redirection NAT/PAT sur ta box pour rediriger le port `22545` externe vers l’IP de la VM proxy.

6. Teste avec l’URL : `http://HomeHomeWCS.webhop.me:22545`.

---
