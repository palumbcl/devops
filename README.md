## 1-1 Why should we run the container with a flag -e to give the environment variables ?
Pour ne pas les avoir en clair dans un fichier comme on l'a actuellement. Cela garantit donc une meilleure sécurité.
## 1-2 Why do we need a volume to be attached to our postgres container?
Afin d'avoir des données persistantes.
## 1-3 Document your database container essentials: commands and Dockerfile.
```
docker run -v pg_data:/var/lib/postgresql/data --net=app-network --name=db palumbcl11/database
```
```
docker build -t palumbcl11/database .
```
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY *.sql /docker-entrypoint-initdb.d/
```
## 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

Ça permet d’avoir une image Docker plus légère en séparant la compilation et l’exécution.

Explication du Dockerfile :

Build 

-On compile l’application avec Maven sans exécuter les tests.

-Le .jar est généré dans target/.

Run

-On récupère uniquement le .jar depuis l’étape de build.

-On exécute l’appli avec java -jar myapp.jar.

## 1-5 Why do we need a reverse proxy ?

Un reverse proxy est utile pour sécuriser les connexions (SSL), répartir la charge entre plusieurs serveurs et centraliser l’authentification au même endroit, au lieu de la gérer sur chaque appli.
## 1-6 Why is docker-compose so important ?

docker-compose est essentiel car il simplifie le déploiement d’un projet en regroupant toute la configuration dans un seul fichier. Il permet de gérer plusieurs services facilement, d’automatiser leur lancement et d’avoir une vue claire de l’ensemble du système.
## 1-7 Document docker-compose most important commands. 

```docker-compose up -d``` : build et run les conteneurs en tâche de fond. 
```docker-compose down -v``` : arrêter et supprimer les conteneurs, ainsi que leurs volumes. 
```docker-compose logs -f``` : afficher les logs. 
```docker-compose ps``` : afficher l'état des conteneurs. 
```docker-compose exec  ... ```: exécuter une commande dans un conteneur en cours d'exécution.
## 1-8 Document your docker-compose file.

```
## Fichier docker-compose.yml
version: "3.7"

## Définition des services
services:
  ## Service backend
  backend:
    ## Image à utiliser
    build:
      context: ./backend
      dockerfile: Dockerfile
    ## Nom du conteneur
    container_name: backend
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2
    ## Dépendances
    depends_on:
      - database
    ## Ports à exposer
    ports:
      - "8080:8080"

  ## Service database
  database:
    ## Image à utiliser
    image: postgres:14.1-alpine
    ## Nom du conteneur
    container_name: db
    ## Variables d'environnement
    environment:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    ## Ports à exposer
    ports:
      - "5432:5432"
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2
    ## Volumes à monter
    volumes:
      - pg_data:/var/lib/postgresql/data

  ## Service frontend
  frontend:
    ## Image à utiliser
    image: httpd:2.4
    container_name: frontend
    ## Ports à exposer
    ports:
      - "80:80"
    ## Volumes à monter
    volumes:
      - ./frontend/index.html:/usr/local/apache2/htdocs/index.html
      - ./frontend/httpd.conf:/usr/local/apache2/conf/httpd.conf
    ## Dépendances
    depends_on:
      - backend
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2

## Réseaux à créer
networks:
  app-network2:
    name: app-network2

## Volumes à créer
volumes:
  pg_data:
    name: pg_data
```
## 1-9 Document your publication commands and published images in dockerhub.
## 1-10 Why do we put our images into an online repo ?
