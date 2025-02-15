# TPs Devops

## TP 1-Docker

## 1-1 Why should we run the container with a flag -e to give the environment variables ?

Pour ne pas les avoir en clair dans un fichier comme on l'a actuellement. Cela garantit donc une meilleure sécurité.

## 1-2 Why do we need a volume to be attached to our postgres container?

Afin d'avoir des données persistantes.

## 1-3 Document your database container essentials: commands and Dockerfile.

```bash
docker run -v pg_data:/var/lib/postgresql/data --net=app-network --name=db palumbcl11/database
```

```bash
docker build -t palumbcl11/database .
```

```dockerfile
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

`docker-compose up -d` : build et run les conteneurs en tâche de fond.
`docker-compose down -v` : arrêter et supprimer les conteneurs, ainsi que leurs volumes.
`docker-compose logs -f` : afficher les logs.
`docker-compose ps` : afficher l'état des conteneurs.
`docker-compose exec  ... `: exécuter une commande dans un conteneur en cours d'exécution.

## 1-8 Document your docker-compose file.

```yaml
## Définition des services
services:
  ## Service backend
  backend:
    ## Image à utiliser
    build:
      context: ./backend
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2
    ## Dépendances
    depends_on:
      - db

  ## Service database
  db:
    env_file: ".env"
    ## Image à utiliser
    build:
      context: ./database
    ## Variables d'environnement
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2
    ## Volumes à monter
    volumes:
      - pg_data:/var/lib/postgresql/data

  ## Service frontend
  frontend:
    ## Image à utiliser
    build:
      context: ./frontend
    ## Ports à exposer
    ports:
      - "80:80"
    ## Dépendances
    depends_on:
      - backend
    ## Réseaux auxquels le conteneur doit être connecté
    networks:
      - app-network2

## Réseaux à créer
networks:
  app-network2:

## Volumes à créer
volumes:
  pg_data:
    name: pg_data
```

## 1-9 Document your publication commands and published images in dockerhub.

```bash
docker tag devops-database palumbcl11/devops-database:1.0
```

```bash
docker push palumbcl11/devops-database:1.0
```

![image](https://github.com/user-attachments/assets/9c95b27e-2590-41f1-ae10-0acd6796fd5d)

## 1-10 Why do we put our images into an online repo ?

Cela permet de gérer différentes version d'une application et d'éviter de les perdre en local. On peut également les récupérer depuis n'importe quel environnement.

## TP 2 -Docker

## 2-1 What are testcontainers ?

Il s'agit simplement de bibliothèques java qui vous permettent d'exécuter un certain nombre de conteneurs Docker pendant les tests.

## 2-2 Document your Github Actions configurations.

```yaml
name: CI devops 2025
on:
  #to begin you want to launch this job in main
  push:
    branches: main
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      #checkout your github code using actions/checkout@v4
      - uses: actions/checkout@v4

      #do the same with another action (actions/setup-java@v4) that enable to setup jdk 21
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: "corretto"
          java-version: "21"

      #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: tp-devops-correction-docker-main/simple-api
```

## 2-3 For what purpose do we need to push docker images ?

Nous poussons des images Docker pour les partager et les déployer sur différents environnements.

## 2-4 Document your quality gate configuration.

```yaml
# Job pour sonar
build:
  name: Build and analyze
  # On utilise la dernière version d'ubuntu
  runs-on: ubuntu-latest
  # On définit les étapes à effectuer
  steps:
    # On commence par récupérer le code source
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0 # Shallow clones should be disabled for a better relevancy of analysis
    - name: Set up JDK 21
      # On utilise l'action setup-java pour installer la version 21 de java
      uses: actions/setup-java@v4
      with:
        java-version: 21
        distribution: "corretto" # Alternative distribution options are available.
    - name: Cache SonarQube packages
      # On utilise l'action cache pour stocker les packages de sonar
      uses: actions/cache@v4
      with:
        path: ~/.sonar/cache
        key: ${{ runner.os }}-sonar
        restore-keys: ${{ runner.os }}-sonar
    - name: Cache Maven packages
      # On utilise l'action cache pour stocker les packages de maven
      uses: actions/cache@v4
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    - name: Build and analyze
      # On lance la commande maven pour construire le projet et lancer l'analyse sonar
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=palumbcl_devops
      working-directory: tp-devops-correction-docker-main/simple-api
```

## 3-1 Document your inventory and base commands

```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts: clement.palumbo.takima.cloud
```

![alt text](image.png)

```bash
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

## 3-2 Document your playbook

```yaml
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker
```

## 3-3 Document your docker_container tasks configuration.

![alt text](image-1.png)

setup.yml:

```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts: clement.palumbo.takima.cloud
```

playbook.yml:

```yaml
- hosts: all
  gather_facts: true
  become: true

  roles:
    - copy_env_file
    - create_network
    - launch_database
    - launch_app
    - launch_proxy
```

.env:

```ini
# psql
POSTGRES_DB=db
POSTGRES_USER=usr
POSTGRES_PASSWORD=pwd
# backend
DATABASE_HOST=db
DATABASE_PASSWORD=pwd
```

```yaml
---
# tasks file for roles/copy_env_file
- name: Copy .env file to the server
  copy:
    src: .env
    dest: /home/admin/.env
```

```yaml
---
# tasks file for roles/create_network
- name: Create app to database network
  docker_network:
    name: app_database_network

- name: Create proxy to app network
  docker_network:
    name: proxy_app_network
```

```yaml
---
# tasks file for roles/launch_app
- name: Launch application container
  docker_container:
    name: simple-api
    image: palumbcl11/tp-devops-backend:latest
    pull: yes
    env_file: /home/admin/.env
    networks:
      - name: app_database_network
      - name: proxy_app_network
```

```yaml
---
# tasks file for roles/launch_database
- name: Launch database container
  docker_container:
    name: db
    image: palumbcl11/tp-devops-db:latest
    state: started
    pull: yes
    env_file: /home/admin/.env
    networks:
      - name: app_database_network
    volumes:
      - pg_data:/var/lib/postgresql/data
```

```yaml
---
# tasks file for roles/launch_proxy
- name: Launch proxy container
  docker_container:
    name: frontend
    image: palumbcl11/tp-devops-http:latest
    state: started
    pull: yes
    ports:
      - "80:80"
    networks:
      - name: proxy_app_network
    restart_policy: always
```
