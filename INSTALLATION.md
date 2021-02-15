# MISE EN PLACE DU SERVEUR RTMP

## Installation avec Docker

Vous pouvez directement lancer le serveur avec l’image construite.
### `docker-compose up --build`

L’image contient la version 1.18 de nginx et la version 1.2.1 du module RTMP de nginx, configurables via les variables d’environnement NGINX_VERSION et NGINX_RTMP_MODULE_VERSION. Les ports 1935 et 8080 sont exposés et le serveur devient accessible via l’adresse http://localhost:8080 ou rtmp://localhost. 
Le fichier de configuration nginx.conf contient les différents endpoints http et l’application rtmp « live » qui possède une sécurité par mot de passe contenu dans le fichier default.conf, par défaut « thomas ».


## Installation manuelle
Vous pouvez également installer le service manuellement. 
1. Mettre à jour l’instance et la liste des sources :
`apt update && apt upgrade -y`


2. Installer nginx, le module rtmp et ffmpeg :
`apt install build-essential libpcre3 libpcre3-dev libssl-dev nginx libnginx-mod-rtmp ffmpeg -y`

3. Ouvrir le fichier de configuration de nginx dans un editeur de texte :
`nano /etc/nginx/nginx.conf`

4. Configurer nginx pour ecouter le flux rtmp entrant :
```
rtmp {
	server {
		listen 1935;
		chunk_size 4096;
		notify_method get;
		
		application live {
			live on;
			
			#Auth to publish stream
			on_publish http://localhost/auth;
     
			#Save a copy of broadcast
			record all;
			record_path /var/www/html/recordings;
			record_unique on;
		}
	}
}
```

Les fichiers entrants seront enregistrés dans le répertoire /var/www/html/recordings du serveur. 

5. Créer le répertoire pour les enregistrements et le rendre accessible en écriture :
`mkdir -p /var/www/html/recordings`
`chown -R www-data:www-data /var/www/html/recordings/`

6. Ouvrir le fichier /etc/nginx/sites-enabled/default, ajouter un block location à la configuration du serveur et Remplacer « MOT DE PASSE » par le mot de passe souhaité :
```
location /auth {
        if ($arg_pwd = 'MOT DE PASSE') {
            return 200;
            }
            return 401;
}
```

7. Redémarrer le service nginx : 
`systemctl restart nginx.service`
