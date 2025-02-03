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
