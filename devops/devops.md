# RAPPORT du TP docker

Thomas PLOIX

## Database

*Why should we run the container with a flag -e to give the environment variables?*

Car ce sont des données sensibles ou des données d’environnement propre à l’utilisateur, elles sont donc privées et ne doivent pas être partager directement dans le Dockerfiles ou autre fichier.

*Why do we need a volume to be attached to our postgres container?*

Cela nous permet de garder nos données de la base de données si jamais le container plante, crash. On pourra relancer le container et réavoir les données sauvegarder, c’est la persitence des données.

Commande utilisée :

docker build -t Tploix/database .

- Création de l’image Tploix/database depuis le dockerfile du répertoire

docker run -e POSTGRES\_DB=db -e POSTGRES\_USER=usr -e POSTGRES\_PASSWORD=pwd --name=database --net=app-network -v /my/own/datadir:/var/lib/postgresql/data Tploix/database

- Lancement du conteneur database depuis l’image Tploix/database, avec des variables d’env, un network et un volume attaché.

docker rm -f database 

- ` `Suppression du conteneur database

Dockerfile :

FROM postgres:14.1-alpine

COPY /docker-entrypoint-initdb.d/\* /docker-entrypoint-initdb.d/


## Backend API

*1-2 Why do we need a multistage build? And explain each step of this dockerfile.*

On utilise un multistage build car on doit le faire en deux fois. Une premiere partie pour Build le projet avec le telechargement de toutes les dépendances maven utilisé par notre projet. Puis on utilise Java JDK, pour compiler le code en Bytecode.

Ensuite la deuxième partie on va utilisé java JRE qui est le Run Time Environnement. Il va récupérer le bytecode qui est le résultat du stage précédent pour le compiler et le lancer.

\# Build
FROM maven:3.8.6-amazoncorretto-17 AS *myapp-build*

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.001.png)*
ENV *MYAPP\_HOME* /opt/myapp

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.002.png)
WORKDIR $*MYAPP\_HOME*

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.003.png)*
COPY pom.xml .

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.004.png)
Run mvn dependency:go-offline

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.005.png)
COPY src ./src

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.006.png)
RUN mvn package -DskipTests

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.007.png)

\# Run
FROM amazoncorretto:17

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.008.png)
ENV *MYAPP\_HOME* /opt/myapp
WORKDIR $*MYAPP\_HOME![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.009.png)*
COPY --from=*myapp-build* $*MYAPP\_HOME*/target/\*.jar $*MYAPP\_HOME*/myapp.jar

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.010.png)

ENTRYPOINT java -jar myapp.jar

![](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.011.png)

<a name="__codelineno-10-8"></a>Commande utilisée :

docker build -t Tploix/backend .

docker run --env-file .env --name=backend --net=app-network -p 8080:8080 Tploix/backend

On utilise un fichier d’environnement qui contient nos variables d’environnement ; On ouvre les ports 8080 :8080 pour communiqué entre le conteneur et nous. On utilise le network pour « établir une ligne de communication » entre le backend et la base de données.
## Frontend
DockerFile :

FROM httpd:2.4

COPY ./public-html/ /usr/local/apache2/htdocs/

COPY proxy.conf /tmp/proxy.conf

RUN cat /tmp/proxy.conf >> /usr/local/apache2/conf/httpd.conf


` `Proxy.conf :

<VirtualHost \*:80>
ServerName localhost

ProxyPreserveHost On
ProxyPass /api/ http://backend:8080/
ProxyPassReverse /api/ http://backend:8080/
</VirtualHost>
LoadModule proxy\_module modules/mod\_proxy.so
LoadModule proxy\_http\_module modules/mod\_proxy\_http.so

*Why do we need a reverse proxy?*

Le Reverse proxy va nous permettre de faire un portail entre le client qui veut accéder à notre application et le front end /backend. Il peut nous permettre d’ajouter des règles de connexion, de mapper des url Spécifiques
##
## Docker-compose
*Why is **docker-compose** so important?*

Le docker compose permet de tout paramétrer dans un seul fichier, de gérer le build des images, les variables d’environnements, les networks et les volumes 

*1-3 Document docker-compose most important commands.*

docker compose up

`	`permet de lancer tous les services qui sont décrit dans le docker-compose.yml

`	`on peut rajouter –build qui va forcé le build des images 

docjer compose down 

`	`permet de kill les instances des conteneurs 

`	`On peut rajouter -V pour supprimer les volumes en même temps

` `*1-4 Document your docker-compose file.*

version: '3.7'

services:
`    `backend:
`        `build: "/backend/."
`        `env\_file:
`            `-   .env  

On utilise les variables d’environnement contenu dans le .env
`        `networks:
`            `- my-network

On connecte avec le network de l’application
`        `depends\_on:
`            `- database

On a besoin que la database soit lancée avant de lancer le backend

`    `database:
`        `build: "/database/."
`        `env\_file:
`            `-   .env
`        `networks:
`            `- my-network
`        `volumes:
`            `-   db-data:/var/lib/postgresql/data

On créer notre volume qu’on a défini plus bas, c’est un named volume

`    `httpd:
`        `build: "/Frontend/."
`        `ports:
`            `- "80:80"
`        `networks:
`            `- my-network
`        `depends\_on:
`            `- backend

networks:
`    `my-network:
On créer le network qui va relier tous nos services entre eux 
volumes:
`    `db-data:

On définit le volume qu’on va utiliser pour notre base de donnée


## Publish

*1-5 Document your publication commands and published images in dockerhub.*

![Une image contenant texte, capture d’écran

Description générée automatiquement](Aspose.Words.8794d491-f06b-45a8-9c68-ac08be3d8735.012.png)

Commandes utilisées : 


Database : 

docker tag Tploix/database tploix/database:1.0

docker push tploix/database:1.0

Backend : 

docker tag Tploix/backend tploix/ backend:1.0

docker push tploix/ backend:1.0

Frontend : 

docker tag Tploix/frontend tploix/ frontend:1.0

docker push tploix/ frontend:1.0

*Why do we put our images into an online repo ?*

En mettant nos images dans un répo en ligne, nous les rendons disponibles et on permet à n’importe qui de pull nos images et les utiliser. On les rends aussi disponibles pour les déployer sur nos serveurs si on en a. 


Bonus :  


Utilisation de fichier d’environnement dans le .env  OK

Utilisation de Volume pour la persistance des données. OK
