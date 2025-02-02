# 🌐 Mise en place d'un serveur web accessible depuis l'extérieur

## 📝 Introduction
Une fois n'est pas coutume, tu vas créer un serveur web sur ta machine locale, et le rendre disponible depuis l’extérieur de ton réseau, sur internet.

---

## ✔️ Étape 1 - Prérequis

Pour cet atelier, tu as besoin :

- 🖥️ Un hyperviseur comme **VirtualBox** pour pouvoir créer des VM.
- 🏗️ **1 VM nommée `webserver`** avec **Debian 12 installé et mis à jour**, avec :
  - 🌐 Une carte réseau en mode **Accès par pont** en **DHCP**.
  - 🔑 Une configuration **SSH fonctionnelle**.
- 🏗️ **1 VM nommée `proxy`** avec **Debian 12 installé et mis à jour**, avec :
  - 🌐 Une carte réseau en mode **Accès par pont** en **DHCP**.
  - 🔑 Une configuration **SSH fonctionnelle**.
- 📶 Une **box opérateur** pour donner accès à Internet.
- 📱 Un périphérique pouvant accéder à internet sans passer par ta box (**smartphone en 4G/5G**).
- 🌍 Un compte sur **No-IP** ([https://my.noip.com/](https://my.noip.com/)).

⚠️ Les tests ont été réalisés sur **Debian 12 avec VirtualBox 7**, exécuté sur **Ubuntu 22.04 LTS**.

---

## 🔬 Étape 2 - Installation d’Apache

Sur la VM `webserver`, exécute :

```bash
apt update && apt upgrade -y
apt install apache2 -y
```

Vérifie que le service Apache est actif :

```bash
systemctl status apache2
```

Trouve l’adresse IP de la carte **Accès par pont** et teste en ouvrant `http://Adresse_IP_privée` sur un navigateur web.

✅ Tu dois voir la page d’accueil par défaut d’Apache.

---

## 🎨 Étape 3 - Personnalisation de la page d'accueil

La page d'accueil par défaut se trouve dans **/var/www/html/index.html**. Modifie son contenu :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Bienvenue sur mon serveur</title>
</head>
<body>
    <h1>Bienvenu sur mon serveur Personnel !</h1>
    <a href="next.html">OK</a>
</body>
</html>
```

Ajoute également **next.html** dans le même dossier avec ce contenu :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Choisissez un site</title>
</head>
<body>
    <h1>Choisissez un site à visiter</h1>
    <a href="https://www.google.com">Google</a>
    <a href="https://www.wikipedia.org">Wikipedia</a>
    <a href="https://www.wildcodeschool.com">WCS</a>
    <a href="index.html">Accueil</a>
</body>
</html>
```

Redémarre Apache pour appliquer les changements :

```bash
systemctl restart apache2
```

Vérifie dans ton navigateur que la page a changé.

---

## 📡 Étape 4 - Configuration de ta box internet

Teste d'abord **depuis ton smartphone en 4G** avec `http://Adresse_IP_Privée_de_ta_VM/`.

❌ Cela ne fonctionne pas, car l’adresse est privée.

Récupère ton **adresse IP publique** en visitant [mon-ip.io](https://mon-ip.io) depuis ton hôte.

Teste ensuite **depuis ton smartphone en 4G** avec `http://Adresse_IP_Publique_de_ta_box/`.

❌ Toujours inaccessible car ta box bloque l’accès. Il faut faire une **redirection de port** :

1. Accède à l'interface d'administration de ta box (ex: `192.168.1.254`).
2. Cherche l'option de **redirection de ports (NAT/PAT)**.
3. **Crée une règle NAT** pour rediriger **le port 80 externe** vers **l'IP privée de ta VM sur le port 80**.
4. Enregistre et teste `http://Adresse_IP_Publique/` sur ton smartphone.

✅ Ton site doit être accessible depuis l’extérieur !

🔒 Pour plus de sécurité, change le **port externe** (ex: `22545`), et teste avec `http://Adresse_IP_Publique:22545`.

---

## 🌍 Étape 5 - Enregistrement d’un nom de domaine

1. **Connecte-toi sur [No-IP](https://my.noip.com/)**.
2. **Crée un nouveau hostname** :
   - 🏷️ **Nom** : HomeHomeWCS
   - 🌎 **Domaine** : webhop.me
   - 📌 **Record Type** : A
   - 🏠 **Adresse IP** : ton IP publique
3. Teste avec `http://HomeHomeWCS.webhop.me:22545` depuis ton smartphone.

✅ Ton site est maintenant accessible via un nom de domaine !

---

## 🔄 Étape 6 - Mise en place d’un reverse proxy

Sur la VM `proxy`, installe Apache et active le module proxy :

```bash
apt install apache2 -y
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_balancer
a2enmod lbmethod_byrequests
systemctl restart apache2
```

Configure le **reverse proxy** en éditant **/etc/apache2/sites-available/000-default.conf** :

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

Ajoute le port d’écoute dans **/etc/apache2/ports.conf** :

```bash
Listen 22545
```

Applique les modifications :

```bash
a2ensite 000-default.conf
systemctl restart apache2
```

🛠️ **Modifie la règle de PAT** pour rediriger **le port 22545 externe** vers **le port 22545 interne** (VM proxy).

Vérifie l’accès à `http://HomeHomeWCS.webhop.me:22545`.

✅ **Ton serveur web est maintenant accessible via un reverse proxy !**

Pour plus de simplicité, modifie **000-default.conf** pour utiliser le **port 80** au lieu du **22545**.

Applique les changements et teste l’accès avec `http://HomeHomeWCS.webhop.me` sans préciser de port.

🎉 **Félicitations ! Ton serveur est maintenant en ligne et sécurisé !**
