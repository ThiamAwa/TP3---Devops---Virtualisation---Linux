# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    # Box de base Ubuntu 20.04
    config.vm.box = "ubuntu/focal64"
    config.vm.box_check_update = false
  
    # ---------------------------
    # Machine : server-back (backend Spring Boot)
    # ---------------------------
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
  
    # ---------------------------
    # Machine : server-dba (MySQL)
    # ---------------------------
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
  
    # ---------------------------
    # Machine : server-front (Nginx + frontend)
    # ---------------------------
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