# 🏗️ TP3 — Architecture 3-Tiers : Vagrant + Spring Boot + MySQL + React/Nginx

![Vagrant](https://img.shields.io/badge/Vagrant-2.x-1563FF?logo=vagrant&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04_LTS-E95420?logo=ubuntu&logoColor=white)
![Java](https://img.shields.io/badge/JDK-17-007396?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-6DB33F?logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![React](https://img.shields.io/badge/React-18.x-61DAFB?logo=react&logoColor=black)
![Nginx](https://img.shields.io/badge/Nginx-1.18-009639?logo=nginx&logoColor=white)
![License](https://img.shields.io/badge/Licence-MIT-brightgreen)

> Déploiement d'une architecture **3-tiers** complète :
> `server-front` (React + Nginx) → `server-back` (Spring Boot) → `server-dba` (MySQL 8).

---

## 📋 Table des matières

- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Structure du projet](#-structure-du-projet)
- [Démarrage rapide](#-démarrage-rapide)
- [Partie 1 — Création des 3 VMs Vagrant](#-partie-1--création-des-3-vms-vagrant)
- [Partie 2 — server-dba : MySQL](#-partie-2--server-dba--installation-mysql)
- [Partie 3 — server-back : JDK 17 & Spring Boot](#-partie-3--server-back--jdk-17--spring-boot)
- [Partie 4 — server-front : React & Nginx](#-partie-4--server-front--react--nginx)
- [Partie 5 — Tests Backend](#-partie-5--tests-backend)
- [Partie 6 — Tests Frontend](#-partie-6--tests-frontend)
- [Résultat final](#-résultat-final)
- [Commandes utiles](#-commandes-utiles)

---

## 🏛️ Architecture

```
                        ┌─────────────────────────────────────────────────────┐
                        │                  Machine Hôte                       │
                        │                                                     │
  Browser               │  ┌──────────────┐   ┌─────────────┐   ┌──────────┐│
  localhost:80 ─────────┼─▶│ server-front │   │ server-back │   │server-dba││
                        │  │192.168.56.30 │   │192.168.56.10│   │192.168.56.20│
                        │  │              │   │             │   │          ││
                        │  │  React 18    │──▶│ Spring Boot │──▶│ MySQL 8  ││
                        │  │  Nginx :80   │   │   :8080     │   │  :3306   ││
                        │  │  /dist       │   │  REST API   │   │  appdb   ││
                        │  └──────────────┘   └─────────────┘   └──────────┘│
                        │                                                     │
                        │  Réseau privé : 192.168.56.0/24                    │
                        └─────────────────────────────────────────────────────┘

  Flux de données :
  [Navigateur] ──HTTP:80──▶ [Nginx/React] ──API:8080──▶ [Spring Boot] ──SQL:3306──▶ [MySQL]
```

---

## 🔧 Prérequis

| Outil | Version | Lien |
|-------|---------|------|
| VirtualBox | 6.x+ | https://www.virtualbox.org/wiki/Downloads |
| Vagrant | 2.x+ | https://developer.hashicorp.com/vagrant/downloads |
| Terminal | — | — |

```bash
vagrant --version    # Vagrant 2.x.x
VBoxManage --version # 6.x.x
```

---

## 📁 Structure du projet

```
tp3/
├── Vagrantfile                          ← Définition des 3 VMs
├── README.md                            ← Ce fichier
│
├── backend/                             ← Application Spring Boot
│   ├── src/
│   │   └── main/
│   │       ├── java/com/tp3/
│   │       │   ├── Tp3Application.java
│   │       │   ├── model/
│   │       │   │   └── Produit.java
│   │       │   ├── repository/
│   │       │   │   └── ProduitRepository.java
│   │       │   └── controller/
│   │       │       └── ProduitController.java
│   │       └── resources/
│   │           └── application.properties
│   ├── pom.xml
│   └── target/
│       └── tp3-backend-1.0.jar          ← JAR exécutable (après build)
│
├── frontend/                            ← Application React
│   ├── src/
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── components/
│   │       └── ProduitList.jsx
│   ├── package.json
│   ├── vite.config.js
│   └── dist/                            ← Build de prod (après npm run build)
│
├── sql/
│   └── init.sql                         ← Script d'initialisation BDD
│
├── nginx/
│   └── tp3.conf                         ← Config Nginx
│
└── screenshots/
    ├── 01_vagrant_up.png
    ├── 02_vagrant_status.png
    ├── 03_ssh_server_dba.png
    ├── 04_mysql_install.png
    ├── 05_mysql_db_create.png
    ├── 06_mysql_user_grant.png
    ├── 07_ssh_server_back.png
    ├── 08_jdk17_version.png
    ├── 09_springboot_start.png
    ├── 10_api_test_curl.png
    ├── 11_ssh_server_front.png
    ├── 12_nginx_status.png
    ├── 13_react_build.png
    ├── 14_app_browser.png
    └── 15_test_final.png
```

---

## ⚡ Démarrage rapide

```bash
# 1. Cloner le dépôt
git clone https://github.com/<votre-username>/tp3-spring-react-mysql.git
cd tp3-spring-react-mysql

# 2. Démarrer les 3 VMs
vagrant up

# 3. Vérifier les statuts
vagrant status

# 4. Accéder aux services
# Frontend  : http://localhost:80
# API REST  : http://localhost:8080/api/produits
```

---

## 🏗️ Partie 1 — Création des 3 VMs Vagrant

### Vagrantfile

```ruby
Vagrant.configure("2") do |config|

  # ─── VM 1 : Backend Spring Boot ─────────────────────────────────
  config.vm.define "server-back" do |back|
    back.vm.box      = "ubuntu/focal64"
    back.vm.hostname = "server-back"

    back.vm.network "private_network", ip: "192.168.56.10"
    back.vm.network "forwarded_port",  guest: 8080, host: 8080

    back.vm.provider "virtualbox" do |vb|
      vb.name   = "server-back"
      vb.memory = "2048"
      vb.cpus   = 2
    end

    back.vm.synced_folder "./backend", "/vagrant/backend"

    back.vm.provision "shell", inline: <<-SHELL
      sudo apt update -q
      sudo apt install -y openjdk-17-jdk maven

      # JAVA_HOME global
      echo 'JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> /etc/environment
      source /etc/environment

      echo "✅ server-back prêt — JDK 17 installé"
    SHELL
  end

  # ─── VM 2 : Base de Données MySQL ───────────────────────────────
  config.vm.define "server-dba" do |dba|
    dba.vm.box      = "ubuntu/focal64"
    dba.vm.hostname = "server-dba"

    dba.vm.network "private_network", ip: "192.168.56.20"

    dba.vm.provider "virtualbox" do |vb|
      vb.name   = "server-dba"
      vb.memory = "1024"
      vb.cpus   = 1
    end

    dba.vm.synced_folder "./sql", "/vagrant/sql"

    dba.vm.provision "shell", inline: <<-SHELL
      sudo apt update -q
      sudo apt install -y mysql-server

      # Autoriser les connexions distantes
      sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
      systemctl restart mysql

      # Initialiser la BDD
      mysql -u root << 'SQL'
CREATE DATABASE IF NOT EXISTS appdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'appuser'@'%' IDENTIFIED BY 'AppPass@2024';
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
USE appdb;
CREATE TABLE IF NOT EXISTS produits (
  id          BIGINT AUTO_INCREMENT PRIMARY KEY,
  nom         VARCHAR(150) NOT NULL,
  description TEXT,
  prix        DECIMAL(10,2) NOT NULL,
  stock       INT DEFAULT 0,
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO produits (nom, description, prix, stock) VALUES
  ('Laptop Pro',    'Ordinateur portable 16Go RAM',  1299.99, 15),
  ('Souris Wireless','Souris sans fil ergonomique',    49.90, 80),
  ('Clavier Mécanique','Cherry MX Brown switches',    129.00, 35);
SQL
      echo "✅ server-dba prêt — MySQL configuré"
    SHELL
  end

  # ─── VM 3 : Frontend React + Nginx ──────────────────────────────
  config.vm.define "server-front" do |front|
    front.vm.box      = "ubuntu/focal64"
    front.vm.hostname = "server-front"

    front.vm.network "private_network", ip: "192.168.56.30"
    front.vm.network "forwarded_port",  guest: 80, host: 80

    front.vm.provider "virtualbox" do |vb|
      vb.name   = "server-front"
      vb.memory = "1024"
      vb.cpus   = 1
    end

    front.vm.synced_folder "./frontend/dist", "/vagrant/dist",
      create: true

    front.vm.provision "shell", inline: <<-SHELL
      sudo apt update -q
      sudo apt install -y nginx nodejs npm

      # Config Nginx
      cat > /etc/nginx/sites-available/tp3 << 'EOF'
server {
    listen 80;
    server_name _;
    root /var/www/tp3;
    index index.html;

    # SPA : redirige tout vers index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API vers server-back
    location /api/ {
        proxy_pass         http://192.168.56.10:8080;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
    }
}
EOF
      ln -sf /etc/nginx/sites-available/tp3 /etc/nginx/sites-enabled/
      rm -f /etc/nginx/sites-enabled/default
      mkdir -p /var/www/tp3

      # Copier le build si présent
      if [ -d /vagrant/dist ] && [ "$(ls -A /vagrant/dist)" ]; then
        cp -r /vagrant/dist/* /var/www/tp3/
      fi

      systemctl restart nginx
      systemctl enable nginx
      echo "✅ server-front prêt — Nginx configuré"
    SHELL
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
Current machine states:

server-back    running (virtualbox)
server-dba     running (virtualbox)
server-front   running (virtualbox)
```

| Capture | Description |
|---------|-------------|
| ![01](./screenshots/01_vagrant_up.png) | `vagrant up` — provisioning des 3 VMs |
| ![02](./screenshots/02_vagrant_status.png) | `vagrant status` — 3 VMs running |

---

## 🗄️ Partie 2 — server-dba : Installation MySQL

### Connexion SSH

```bash
vagrant ssh server-dba
# vagrant@server-dba:~$
```

| Capture | Description |
|---------|-------------|
| ![03](./screenshots/03_ssh_server_dba.png) | Connexion SSH à server-dba |

### Vérification MySQL

```bash
sudo systemctl status mysql
# ● mysql.service - MySQL Community Server
#    Active: active (running)

mysql --version
# mysql  Ver 8.0.xx Distrib 8.0.xx, for Linux (x86_64)
```

| Capture | Description |
|---------|-------------|
| ![04](./screenshots/04_mysql_install.png) | MySQL 8.0 — active (running) |

### Vérification de la base et de l'utilisateur

```bash
sudo mysql -u root
```

```sql
SHOW DATABASES;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | appdb              |
-- | information_schema |
-- | mysql              |
-- +--------------------+

SELECT User, Host FROM mysql.user WHERE User = 'appuser';
-- +---------+------+
-- | User    | Host |
-- +---------+------+
-- | appuser | %    |
-- +---------+------+

USE appdb;
SELECT * FROM produits;
-- +----+------------------+--------------------------------+---------+-------+
-- | id | nom              | description                    | prix    | stock |
-- +----+------------------+--------------------------------+---------+-------+
-- |  1 | Laptop Pro       | Ordinateur portable 16Go RAM   | 1299.99 |    15 |
-- |  2 | Souris Wireless  | Souris sans fil ergonomique    |   49.90 |    80 |
-- |  3 | Clavier Mécanique| Cherry MX Brown switches       |  129.00 |    35 |
-- +----+------------------+--------------------------------+---------+-------+
```

| Capture | Description |
|---------|-------------|
| ![05](./screenshots/05_mysql_db_create.png) | BDD `appdb` et table `produits` |
| ![06](./screenshots/06_mysql_user_grant.png) | Utilisateur `appuser` avec droits sur `appdb` |

---

## ☕ Partie 3 — server-back : JDK 17 & Spring Boot

### Connexion SSH

```bash
vagrant ssh server-back
# vagrant@server-back:~$
```

### Vérification JDK 17

```bash
java -version
# openjdk version "17.x.xx" 2023-xx-xx
# OpenJDK Runtime Environment (build 17.x.xx)
# OpenJDK 64-Bit Server VM (build 17.x.xx, mixed mode, sharing)

echo $JAVA_HOME
# /usr/lib/jvm/java-17-openjdk-amd64
```

| Capture | Description |
|---------|-------------|
| ![07](./screenshots/07_ssh_server_back.png) | Connexion SSH à server-back |
| ![08](./screenshots/08_jdk17_version.png) | `java -version` — OpenJDK 17 confirmé |

---

### Application Spring Boot

#### `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
  </parent>

  <groupId>com.tp3</groupId>
  <artifactId>tp3-backend</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>

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
    <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
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
# ── Serveur ──────────────────────────────────────────
server.port=8080

# ── Base de données (server-dba) ─────────────────────
spring.datasource.url=jdbc:mysql://192.168.56.20:3306/appdb?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=appuser
spring.datasource.password=AppPass@2024
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ── JPA / Hibernate ───────────────────────────────────
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# ── CORS (autoriser server-front) ─────────────────────
app.cors.allowed-origins=http://192.168.56.30,http://localhost
```

#### `Produit.java` — Entité JPA

```java
package com.tp3.model;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name = "produits")
public class Produit {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nom;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal prix;

    private Integer stock;

    // ─── Constructeurs ──────────────────────────────
    public Produit() {}

    public Produit(String nom, String description, BigDecimal prix, Integer stock) {
        this.nom = nom; this.description = description;
        this.prix = prix; this.stock = stock;
    }

    // ─── Getters & Setters ──────────────────────────
    public Long       getId()          { return id; }
    public String     getNom()         { return nom; }
    public String     getDescription() { return description; }
    public BigDecimal getPrix()        { return prix; }
    public Integer    getStock()       { return stock; }

    public void setId(Long id)                    { this.id = id; }
    public void setNom(String nom)                { this.nom = nom; }
    public void setDescription(String desc)       { this.description = desc; }
    public void setPrix(BigDecimal prix)          { this.prix = prix; }
    public void setStock(Integer stock)           { this.stock = stock; }
}
```

#### `ProduitRepository.java`

```java
package com.tp3.repository;

import com.tp3.model.Produit;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    List<Produit> findByNomContainingIgnoreCase(String nom);
}
```

#### `ProduitController.java` — REST API

```java
package com.tp3.controller;

import com.tp3.model.Produit;
import com.tp3.repository.ProduitRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/produits")
@CrossOrigin(origins = {"http://192.168.56.30", "http://localhost"})
public class ProduitController {

    private final ProduitRepository repo;

    public ProduitController(ProduitRepository repo) {
        this.repo = repo;
    }

    // GET /api/produits
    @GetMapping
    public List<Produit> getAll() {
        return repo.findAll();
    }

    // GET /api/produits/{id}
    @GetMapping("/{id}")
    public ResponseEntity<Produit> getById(@PathVariable Long id) {
        return repo.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // POST /api/produits
    @PostMapping
    public Produit create(@RequestBody Produit p) {
        return repo.save(p);
    }

    // PUT /api/produits/{id}
    @PutMapping("/{id}")
    public ResponseEntity<Produit> update(@PathVariable Long id, @RequestBody Produit p) {
        return repo.findById(id).map(existing -> {
            existing.setNom(p.getNom());
            existing.setDescription(p.getDescription());
            existing.setPrix(p.getPrix());
            existing.setStock(p.getStock());
            return ResponseEntity.ok(repo.save(existing));
        }).orElse(ResponseEntity.notFound().build());
    }

    // DELETE /api/produits/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (!repo.existsById(id)) return ResponseEntity.notFound().build();
        repo.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

#### `Tp3Application.java`

```java
package com.tp3;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Tp3Application {
    public static void main(String[] args) {
        SpringApplication.run(Tp3Application.class, args);
    }
}
```

### Build et démarrage

```bash
# Depuis server-back
cd /vagrant/backend
mvn clean package -DskipTests

# Lancer l'application en arrière-plan
nohup java -jar target/tp3-backend-1.0.jar > /var/log/tp3-backend.log 2>&1 &

# Vérifier le démarrage
tail -f /var/log/tp3-backend.log
# ...Tomcat started on port 8080...
# ...Started Tp3Application in 4.x seconds...

# Test local
curl http://localhost:8080/api/produits
```

### Service systemd (optionnel)

```bash
sudo tee /etc/systemd/system/tp3-backend.service > /dev/null << 'EOF'
[Unit]
Description=TP3 Spring Boot Backend
After=network.target

[Service]
User=vagrant
WorkingDirectory=/vagrant/backend
ExecStart=/usr/bin/java -jar /vagrant/backend/target/tp3-backend-1.0.jar
SuccessExitStatus=143
Restart=always
StandardOutput=append:/var/log/tp3-backend.log
StandardError=append:/var/log/tp3-backend.log

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable tp3-backend
sudo systemctl start tp3-backend
```

| Capture | Description |
|---------|-------------|
| ![09](./screenshots/09_springboot_start.png) | Spring Boot démarré — port 8080 |
| ![10](./screenshots/10_api_test_curl.png) | `curl /api/produits` — réponse JSON |

---

## 🌐 Partie 4 — server-front : React & Nginx

### Connexion SSH

```bash
vagrant ssh server-front
# vagrant@server-front:~$
```

### Vérification Nginx

```bash
sudo systemctl status nginx
# ● nginx.service - A high performance web server
#    Active: active (running)

nginx -v
# nginx version: nginx/1.18.x
```

| Capture | Description |
|---------|-------------|
| ![11](./screenshots/11_ssh_server_front.png) | Connexion SSH à server-front |
| ![12](./screenshots/12_nginx_status.png) | Nginx — active (running) |

---

### Application React

#### `package.json`

```json
{
  "name": "tp3-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev":   "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react":     "^18.2.0",
    "react-dom": "^18.2.0",
    "axios":     "^1.6.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.0",
    "vite": "^5.0.0"
  }
}
```

#### `vite.config.js`

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://192.168.56.10:8080',
        changeOrigin: true
      }
    }
  }
})
```

#### `src/main.jsx`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

#### `src/App.jsx`

```jsx
import { useState } from 'react'
import ProduitList from './components/ProduitList'
import './App.css'

export default function App() {
  return (
    <div className="app">
      <header>
        <h1>🛒 Gestion des Produits</h1>
        <p>server-front → server-back → server-dba</p>
      </header>
      <main>
        <ProduitList />
      </main>
    </div>
  )
}
```

#### `src/components/ProduitList.jsx`

```jsx
import { useEffect, useState } from 'react'
import axios from 'axios'

const API = '/api/produits'

export default function ProduitList() {
  const [produits, setProduits] = useState([])
  const [form, setForm]         = useState({ nom:'', description:'', prix:'', stock:'' })
  const [loading, setLoading]   = useState(true)
  const [error, setError]       = useState(null)

  const fetchProduits = async () => {
    try {
      setLoading(true)
      const res = await axios.get(API)
      setProduits(res.data)
    } catch (e) {
      setError('Impossible de contacter le backend (192.168.56.10:8080)')
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => { fetchProduits() }, [])

  const handleSubmit = async (e) => {
    e.preventDefault()
    await axios.post(API, { ...form, prix: parseFloat(form.prix), stock: parseInt(form.stock) })
    setForm({ nom:'', description:'', prix:'', stock:'' })
    fetchProduits()
  }

  const handleDelete = async (id) => {
    if (!confirm('Supprimer ce produit ?')) return
    await axios.delete(`${API}/${id}`)
    fetchProduits()
  }

  if (loading) return <p className="loading">Chargement...</p>
  if (error)   return <p className="error">{error}</p>

  return (
    <div>
      {/* ─── Tableau des produits ────────────────── */}
      <table>
        <thead>
          <tr>
            <th>ID</th><th>Nom</th><th>Description</th>
            <th>Prix (€)</th><th>Stock</th><th>Action</th>
          </tr>
        </thead>
        <tbody>
          {produits.map(p => (
            <tr key={p.id}>
              <td>{p.id}</td>
              <td>{p.nom}</td>
              <td>{p.description}</td>
              <td>{parseFloat(p.prix).toFixed(2)}</td>
              <td>{p.stock}</td>
              <td>
                <button className="btn-delete" onClick={() => handleDelete(p.id)}>
                  Supprimer
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      {/* ─── Formulaire ajout ────────────────────── */}
      <form onSubmit={handleSubmit} className="form-add">
        <h3>Ajouter un produit</h3>
        <input required placeholder="Nom"
          value={form.nom} onChange={e => setForm({...form, nom: e.target.value})} />
        <input placeholder="Description"
          value={form.description} onChange={e => setForm({...form, description: e.target.value})} />
        <input required type="number" step="0.01" placeholder="Prix"
          value={form.prix} onChange={e => setForm({...form, prix: e.target.value})} />
        <input required type="number" placeholder="Stock"
          value={form.stock} onChange={e => setForm({...form, stock: e.target.value})} />
        <button type="submit" className="btn-add">➕ Ajouter</button>
      </form>
    </div>
  )
}
```

#### `src/App.css`

```css
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: Arial, sans-serif; background: #f5f7fa; color: #333; }

.app { max-width: 960px; margin: 0 auto; padding: 20px; }

header { background: #2E75B6; color: white; padding: 24px; border-radius: 8px; margin-bottom: 24px; }
header h1 { font-size: 1.6rem; }
header p  { font-size: 0.9rem; opacity: 0.8; margin-top: 6px; }

table { width: 100%; border-collapse: collapse; background: white;
        border-radius: 8px; overflow: hidden; box-shadow: 0 1px 4px rgba(0,0,0,0.1); }
th { background: #2E75B6; color: white; padding: 12px 14px; text-align: left; }
td { padding: 10px 14px; border-bottom: 1px solid #eee; }
tr:last-child td { border-bottom: none; }
tr:hover td { background: #f0f6ff; }

.btn-delete { background: #dc3545; color: white; border: none; padding: 6px 12px;
              border-radius: 4px; cursor: pointer; }
.btn-delete:hover { background: #b02a37; }

.form-add { background: white; padding: 24px; border-radius: 8px; margin-top: 24px;
            box-shadow: 0 1px 4px rgba(0,0,0,0.1); }
.form-add h3 { margin-bottom: 16px; color: #2E75B6; }
.form-add input { display: block; width: 100%; padding: 9px 12px; margin-bottom: 10px;
                  border: 1px solid #ccc; border-radius: 4px; font-size: 14px; }
.btn-add { background: #2E75B6; color: white; border: none; padding: 10px 24px;
           border-radius: 4px; cursor: pointer; font-size: 15px; }
.btn-add:hover { background: #1a5c9a; }

.loading { text-align: center; padding: 40px; font-size: 1.2rem; color: #888; }
.error   { color: #dc3545; background: #fde8e8; padding: 16px; border-radius: 8px; margin: 20px 0; }
```

### Build et déploiement

```bash
# Sur la machine hôte (ou dans server-front via dossier partagé)
cd frontend
npm install
npm run build
# → génère frontend/dist/

# Sur server-front — copier le build
sudo cp -r /vagrant/dist/* /var/www/tp3/
sudo systemctl reload nginx
```

### Configuration Nginx — `nginx/tp3.conf`

```nginx
server {
    listen 80;
    server_name _;

    # Fichiers statiques React
    root  /var/www/tp3;
    index index.html;

    # SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy vers l'API Spring Boot
    location /api/ {
        proxy_pass         http://192.168.56.10:8080;
        proxy_http_version 1.1;
        proxy_set_header   Host            $host;
        proxy_set_header   X-Real-IP       $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

| Capture | Description |
|---------|-------------|
| ![13](./screenshots/13_react_build.png) | `npm run build` — génération du dossier `dist/` |
| ![14](./screenshots/14_app_browser.png) | Application React dans le navigateur (localhost:80) |

---

## 🧪 Partie 5 — Tests Backend

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

### Test 3 — Endpoints REST (CRUD complet)

```bash
BASE="http://localhost:8080/api/produits"

# GET tous les produits
echo "── GET ALL ──"
curl -s $BASE | python3 -m json.tool

# GET un produit par ID
echo "── GET BY ID ──"
curl -s $BASE/1 | python3 -m json.tool

# POST — créer un produit
echo "── POST ──"
curl -s -X POST $BASE \
  -H "Content-Type: application/json" \
  -d '{"nom":"Écran 4K","description":"27 pouces IPS","prix":549.99,"stock":20}' \
  | python3 -m json.tool

# PUT — modifier un produit
echo "── PUT ──"
curl -s -X PUT $BASE/1 \
  -H "Content-Type: application/json" \
  -d '{"nom":"Laptop Pro MAX","description":"32Go RAM","prix":1599.99,"stock":10}' \
  | python3 -m json.tool

# DELETE — supprimer
echo "── DELETE ──"
curl -s -X DELETE $BASE/4 -w "HTTP %{http_code}\n"
```

Résultats attendus :

```
── GET ALL ──
[
  {"id":1,"nom":"Laptop Pro","description":"...","prix":1299.99,"stock":15},
  {"id":2,"nom":"Souris Wireless","description":"...","prix":49.90,"stock":80},
  {"id":3,"nom":"Clavier Mécanique","description":"...","prix":129.00,"stock":35}
]
── POST ──
{"id":4,"nom":"Écran 4K","description":"27 pouces IPS","prix":549.99,"stock":20}
── DELETE ──
HTTP 204
```

### Test 4 — Vérification en BDD

```bash
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT COUNT(*) AS total FROM produits;"
```

| Capture | Description |
|---------|-------------|
| ![10](./screenshots/10_api_test_curl.png) | Tous les tests CRUD réussis |

---

## 🧪 Partie 6 — Tests Frontend

### Test 1 — Nginx accessible

```bash
# Depuis server-front
curl -I http://localhost
# HTTP/1.1 200 OK
# Server: nginx/1.18.x
# Content-Type: text/html

# Depuis la machine hôte
curl -I http://localhost:80
# HTTP/1.1 200 OK
```

### Test 2 — Proxy API fonctionnel

```bash
# Appel API via le proxy Nginx (server-front → server-back)
curl -s http://localhost/api/produits | python3 -m json.tool
# Doit retourner la liste JSON des produits
```

### Test 3 — Vérification des fichiers React

```bash
ls /var/www/tp3/
# index.html   assets/   vite.svg

cat /etc/nginx/sites-enabled/tp3
# Vérifie la config Nginx active

sudo nginx -t
# nginx: configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Test 4 — Test de bout en bout (End-to-End)

```bash
# 1. Ajouter un produit via l'API (simule le formulaire React)
curl -s -X POST http://localhost/api/produits \
  -H "Content-Type: application/json" \
  -d '{"nom":"Test E2E","description":"Produit de test","prix":9.99,"stock":1}'

# 2. Vérifier dans React (via Nginx)
curl -s http://localhost/api/produits | grep "Test E2E"
# {"id":5,"nom":"Test E2E",...}

# 3. Vérifier directement en base
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb \
  -e "SELECT * FROM produits WHERE nom='Test E2E';"
```

### Test 5 — Vérification des 3 services

```bash
# server-back
vagrant ssh server-back -c "sudo systemctl status tp3-backend --no-pager"

# server-dba
vagrant ssh server-dba -c "sudo systemctl status mysql --no-pager"

# server-front
vagrant ssh server-front -c "sudo systemctl status nginx --no-pager"
```

Résultats attendus :

```
● tp3-backend.service - TP3 Spring Boot Backend
   Active: active (running)

● mysql.service - MySQL Community Server
   Active: active (running)

● nginx.service - A high performance web server
   Active: active (running)
```

| Capture | Description |
|---------|-------------|
| ![15](./screenshots/15_test_final.png) | Test E2E — les 3 services actifs, données cohérentes |

---

## ✅ Résultat final

| Composant | IP | Service | Port | URL |
|-----------|----|---------|------|-----|
| server-back | `192.168.56.10` | Spring Boot | `8080` | `http://localhost:8080/api/produits` |
| server-dba  | `192.168.56.20` | MySQL 8 | `3306` | `appdb` / `appuser` |
| server-front| `192.168.56.30` | Nginx + React | `80` | `http://localhost` |

| Variable | Valeur |
|----------|--------|
| JDK | OpenJDK 17 |
| Framework backend | Spring Boot 3.x |
| BDD | `appdb` — table `produits` |
| Utilisateur BDD | `appuser` / `AppPass@2024` |
| Framework frontend | React 18 + Vite |
| Serveur web | Nginx 1.18 |

---

## 🛠️ Commandes utiles

```bash
# ── Vagrant ──────────────────────────────────────────────────────
vagrant up                         # Démarrer les 3 VMs
vagrant up server-back             # Démarrer une VM spécifique
vagrant halt                       # Éteindre toutes les VMs
vagrant ssh server-back            # SSH sur server-back
vagrant ssh server-dba             # SSH sur server-dba
vagrant ssh server-front           # SSH sur server-front
vagrant reload --provision         # Reprovisionner
vagrant destroy -f                 # Supprimer toutes les VMs

# ── Backend Spring Boot ──────────────────────────────────────────
cd /vagrant/backend && mvn clean package -DskipTests
java -jar target/tp3-backend-1.0.jar
sudo systemctl start|stop|status tp3-backend
tail -f /var/log/tp3-backend.log

# ── MySQL ────────────────────────────────────────────────────────
sudo systemctl start|stop|status mysql
mysql -u appuser -p'AppPass@2024' -h 192.168.56.20 appdb
mysql -u root

# ── Nginx + React ────────────────────────────────────────────────
sudo systemctl start|stop|status nginx
sudo nginx -t
sudo nginx -s reload
ls /var/www/tp3/

# ── Frontend (machine hôte) ──────────────────────────────────────
cd frontend && npm install && npm run build
cp -r dist/* ../frontend/dist/    # Dossier partagé Vagrant
```

---

## 📄 Licence

Distribué sous licence MIT.

---

*TP3 réalisé dans le cadre du cours DevOps / Administration Système*
