#  TP3 — Architecture 3-Tiers : Vagrant + Spring Boot + MySQL + Angular/Nginx

![Vagrant](https://img.shields.io/badge/Vagrant-2.x-1563FF?logo=vagrant&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04_LTS-E95420?logo=ubuntu&logoColor=white)
![Java](https://img.shields.io/badge/JDK-17-007396?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-6DB33F?logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![Angular](https://img.shields.io/badge/Angular-17.x-DD0031?logo=angular&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-1.18-009639?logo=nginx&logoColor=white)
![License](https://img.shields.io/badge/Licence-MIT-brightgreen)

> Déploiement d'une architecture **3-tiers** complète :
> `server-front` (Angular + Nginx) → `server-back` (Spring Boot) → `server-dba` (MySQL 8)

```
Flux de données :
[Navigateur] ──HTTP:80──▶ [Nginx/Angular] ──API:8080──▶ [Spring Boot] ──SQL:3306──▶ [MySQL]
```

---

##  Structure du projet

![20](20.png) 
![21](21.png) 

---

## Création des 3 VMs Vagrant

### Vagrantfile

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    # Box de base Ubuntu 20.04
    config.vm.box = "ubuntu/focal64"
    config.vm.box_check_update = false
  
    # Machine : server-back (backend Spring Boot)
 
    config.vm.define "server-back" do |back|
      back.vm.hostname = "server-back"
      back.vm.network "private_network", ip: "192.168.56.10"
      # Redirection du port pour accéder à l'application (si elle tourne sur 8080)
      back.vm.network "forwarded_port", guest: 8080, host: 8080
      back.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
        vb.name = "server-back"
      end
      # Dossier partagé pour le projet backend (optionnel)
      back.vm.synced_folder "./backend", "/vagrant/backend"
    end
  

    # Machine : server-dba (MySQL)
  
    config.vm.define "server-dba" do |dba|
      dba.vm.hostname = "server-dba"
      dba.vm.network "private_network", ip: "192.168.56.11"
      # Redirection du port MySQL (optionnel, pour accès depuis l'hôte)
      dba.vm.network "forwarded_port", guest: 3306, host: 3306
      dba.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = 1
        vb.name = "server-dba"
      end
    end
  
   
    # Machine : server-front (Nginx + frontend)
   
    config.vm.define "server-front" do |front|
      front.vm.hostname = "server-front"
      front.vm.network "private_network", ip: "192.168.56.12"
      # Redirection du port 80 pour accéder au frontend
      front.vm.network "forwarded_port", guest: 80, host: 8081
      front.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = 1
        vb.name = "server-front"
      end
      # Dossier partagé pour le projet frontend (build)
      # front.vm.synced_folder "./frontend", "/vagrant/frontend"
      front.vm.synced_folder "./frontend/dist", "/vagrant/frontend-dist"
    end
  end
    
```

### Démarrage

```bash
vagrant up
vagrant status
```

Résultat attendu :

```
server-back    running (virtualbox)
server-dba     running (virtualbox)
server-front   running (virtualbox)
```

| Capture | Description |
|---------|-------------|
| ![01](1.png) | `vagrant up` — provisioning des 3 VMs |


---

##  server-dba : Installation MySQL

### Connexion SSH à server-dba 

 ![02](2.png) 

### Vérification MySQL

## Installation de MySQL
 ![04](3.png)
 
## Éditez le fichier de configuration
![04](4.png)

## Création de la base
![05](5.png)

## Installer JDK 17 et Maven

![06](6.png)
## sudo apt install openjdk-17-jdk maven -y-Installer JDK 17 et Maven
![07](7.png)

## Compiler et lancer l'application
![08](8.png)
![09](9.png)
![10](10.png)

---

### Application Spring Boot

#### `pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/>
    </parent>

    <groupId>sn.dev</groupId>
    <artifactId>crudProduit</artifactId>
    <version>0.0.1-SNAPSHOT</version>
   
    <packaging>jar</packaging>
    <name>crudProduit</name>
    <description>Application Spring Boot avec MySQL (backend autonome)</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Driver MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
      
        <finalName>crudProduit</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### `application.properties`

```properties
spring.application.name=crudProduit
spring.datasource.url=jdbc:mysql://192.168.56.11:3306/back_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=backuser
spring.datasource.password=password123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect

spring.thymeleaf.cache=false

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

#### `Structure du projet back

![22](22.png)
    

## server-front : Angular & Nginx

### Connexion SSH

![11](11.png)

### Vérification Nginx

## Installation de Nginx et redemarrer Nginx

![16](16.png)
![17](17.png)
![13](avant18.png)
![18](18.png)


---

### Application Angular



### Build et déploiement

```bash
# Sur la machine hôte
cd frontend
npm install
 ng build --configuration=production
# → génère frontend/dist/

# Sur server-front
sudo cp -r /vagrant/dist/* /var/www/tp3/
sudo systemctl reload nginx
```

### Configuration Nginx — `nginx/tp3.conf`

```nginx
server {
    listen 80;
    server_name _;
    root  /var/www/tp3;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass         http://192.168.56.10:8080;
        proxy_http_version 1.1;
        proxy_set_header   Host            $host;
        proxy_set_header   X-Real-IP       $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```



---

## Partie 5 — Tests Backend

### Test 1 — Ping inter-VMs

```bash
# Depuis server-back
ping -c 3 192.168.56.20   # → server-dba
ping -c 3 192.168.56.30   # → server-front
```

### Test 2 — Connexion MySQL depuis server-back

```bash
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT * FROM produits;"
```




### Test 4 — Vérification en BDD

```bash
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT COUNT(*) AS total FROM produits;"
```

| Capture | Description |
|---------|-------------|
| ![10](33.png) | Tous les tests CRUD réussis |

---

## Partie 6 — Tests Frontend

### Test 1 — Nginx accessible

```bash
curl -I http://localhost
# HTTP/1.1 200 OK
# Server: nginx/1.18.x
```

### Test 2 — Proxy API fonctionnel

```bash
# Appel API via le proxy Nginx (server-front → server-back)
curl -s http://localhost/api/produits | python3 -m json.tool
```

### Test 3 — Vérification des fichiers React

```bash
ls /var/www/tp3/
# index.html   assets/

sudo nginx -t
# nginx: configuration file syntax is ok
# nginx: configuration file test is successful
```

### Test 4 — Test de bout en bout (End-to-End)

```bash
# 1. Ajouter un produit via l'API
curl -s -X POST http://localhost/api/produits \
  -H "Content-Type: application/json" \
  -d '{"nom":"Test E2E","description":"Produit de test","prix":9.99,"stock":1}'

# 2. Vérifier via Nginx
curl -s http://localhost/api/produits | grep "Test E2E"

# 3. Vérifier directement en base
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT * FROM produits WHERE nom='Test E2E';"
```

### Test 5 — Vérification des 3 services

```bash
vagrant ssh server-back  -c "sudo systemctl status tp3-backend --no-pager"
vagrant ssh server-dba   -c "sudo systemctl status mysql --no-pager"
vagrant ssh server-front -c "sudo systemctl status nginx --no-pager"
```

Résultats attendus :

```
● tp3-backend.service - TP3 Spring Boot Backend   Active: active (running)
● mysql.service - MySQL Community Server           Active: active (running)
● nginx.service - A high performance web server    Active: active (running)
```



---

##  Résultat final

| Composant | IP | Service | Port | URL |
|-----------|----|---------|------|-----|
| server-back  | `192.168.56.10` | Spring Boot | `8080` | `http://localhost:8080/api/produits` |
| server-dba   | `192.168.56.20` | MySQL 8     | `3306` | `appdb` / `appuser` |
| server-front | `192.168.56.30` | Nginx + React | `80` | `http://localhost` |

| Variable | Valeur |
|----------|--------|
| JDK | OpenJDK 17 |
| Framework backend | Spring Boot 3.x |
| BDD | `appdb` — table `produits` |
| Utilisateur BDD | `appuser` / `AppPass@2024` |
| Framework frontend | Angular + Vite |
| Serveur web | Nginx 1.18 |

---

##  Commandes utiles

```bash
# ── Vagrant ──────────────────────────────────────────────────────
vagrant up                          # Démarrer les 3 VMs
vagrant up server-back              # Démarrer une VM spécifique
vagrant halt                        # Éteindre toutes les VMs
vagrant ssh server-back             # SSH sur server-back
vagrant ssh server-dba              # SSH sur server-dba
vagrant ssh server-front            # SSH sur server-front
vagrant reload --provision          # Reprovisionner
vagrant destroy -f                  # Supprimer toutes les VMs

# ── Backend Spring Boot ──────────────────────────────────────────
cd /vagrant/backend && mvn clean package -DskipTests
java -jar target/tp3-backend-1.0.jar
sudo systemctl start|stop|status tp3-backend
tail -f /var/log/tp3-backend.log

# ── MySQL ────────────────────────────────────────────────────────
sudo systemctl start|stop|status mysql
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb
sudo mysql -u root

# ── Nginx + React ────────────────────────────────────────────────
sudo systemctl start|stop|status nginx
sudo nginx -t
sudo nginx -s reload
ls /var/www/tp3/

# ── Frontend (machine hôte) ──────────────────────────────────────
cd frontend && npm install && npm run build
```

---

##  Licence

Distribué sous licence MIT.

---

*TP3 réalisé dans le cadre du cours DevOps / Administration Système*
