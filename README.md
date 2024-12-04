# Tutorial Jenkins - Despliegue de un Sistema de CI/CD en Microsoft Azure

¡Bienvenido al Tutorial Jenkins!

Esta guía te llevará paso a paso a través de la implementación de un robusto sistema de Integración Continua y Despliegue Continuo (CI/CD) utilizando Jenkins y la nube de Microsoft Azure. Este tutorial está especialmente diseñado para aquellos que ya cuentan con conocimientos fundamentales en Git, GitHub, Kubernetes, Trivy, SonarQube, ArgoCD, React, NPM, Node, Docker y DockerHub, así como experiencia en el manejo de la terminal de Ubuntu y entornos en la nube de Azure.

## Contenido del Tutorial

1. [Iniciando Máquinas Virtuales en Azure](#iniciando-máquinas-virtuales-en-azure)
2. [Creando Jenkins-Master VM](#creando-jenkins-master-vm)
3. [Creando Jenkins-Agent VM](#creando-jenkins-agent-vm)
4. [Creando SonarQube VM](#creando-sonarqube-vm)
5. [Generación de Claves SSH](#generación-de-claves-ssh)
6. [Accediendo a Jenkins](#accediendo-a-jenkins)
7. [Configurando Jenkins](#configurando-jenkins)
   - [Configurando Nodos](#configurando-nodos)
   - [Instalando Plugins](#instalando-plugins)
   - [Configurando Plugins](#configurando-plugins)
   - [Conectando Jenkins con SonarQube](#conectando-jenkins-con-sonarqube)
   - [Conectando Jenkins con mi Cuenta de GitHub](#conectando-jenkins-con-mi-cuenta-de-github)
   - [Conectando Jenkins con mi Cuenta de DockerHub](#conectando-jenkins-con-mi-cuenta-de-dockerhub)
8. [Creando Pipeline y Jenkinsfile](#creando-pipeline-y-jenkinsfile)
9. [Creando AKS Cluster](#creando-aks-cluster)
   - [Instalando Azure-cli (az)](#instalando-azure-cli-az)
   - [Iniciando Sesión en tu Cuenta de Azure](#iniciando-sesión-en-tu-cuenta-de-azure)
10. [Creando ArgoCD](#creando-argocd)
    - [Instalación Manual](#instalación-manual)
    - [Instalación con Helm](#instalación-con-helm)
      - [Usando Repositorio Remoto](#usando-repositorio-remoto)
      - [Usando un Chart Local](#usando-un-chart-local)
    - [Asignando Cluster AKS a ArgoCD](#asignando-cluster-aks-a-argocd)
11. [Creando Proyecto Gitops](#creando-proyecto-gitops)
    - [Conectando con el Repositorio Gitops de GitHub](#conectando-con-el-repositorio-gitops-de-github)
    - [Creando Aplicación de Monitoreo en Argocd](#creando-aplicación-de-monitoreo-en-argocd)
    - [Creando Aplicación y Proyecto en ArgoCD desde Manifiestos](#creando-aplicación-y-proyecto-en-argocd-desde-manifiestos)
    - [Creando Pipeline para CD](#creando-pipeline-para-cd)
    - [Creando Jenkinsfile para CD](#creando-jenkinsfile-para-cd)
12. [Conectando Proyecto CI con Proyecto CD](#conectando-proyecto-ci-con-proyecto-cd)
13. [Conectando Webhook de GitHub](#conectando-webhook-de-github)
14. [Conectando con Gmail](#conectando-con-gmail)
15. [Usando Image Updater](#usando-image-updater)

## Arquitectura del Sistema

La siguiente imagen representa la arquitectura del sistema que implementaremos en este tutorial.

¡Explora la imagen para comprender mejor la interacción entre estos componentes y sigue los pasos del tutorial para implementar este sistema de CI/CD en tu entorno Azure!

![Arquitecura React Demo GitOps](./react-demo-gitops.svg)

Este tutorial te llevará paso a paso desde la configuración inicial hasta la implementación de un flujo de CI/CD robusto utilizando las tecnologías mencionadas. ¡Comencemos!

## Iniciando Máquinas Virtuales en Azure

Lo primero crear tres VM con el grupo de recursos `jenkins_group`.

Una máquina virtual se llamará "Jenkins-Master", la otra "Jenkins-Agent" y la última máquina virtual se llamará "SonarQube".

En Jenkins-Master abrimos el puerto 8080 TCP con el nombre JenkinsPort.

En SonarQube abrimos el puerto TCP 9000.

## Creando Jenkins-Master VM

- Crear una VM DS1_v2 con 1CPUs y 3.5GiB de memoria RAM y 15GB o más de almacenamiento.

```
sudo chmod 700 Jenkins-Master_key.pem
```

```
ssh -i <Key-pair .pem> azureuser@<Public IP>
```

```
sudo apt update
```

```
sudo apt upgrade
```

```
sudo init 6
```

```
sudo apt install openjdk-17-jre-headless
```

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

```
sudo systemctl enable jenkins
```

```
sudo systemctl start jenkins
```

```
systemctl status jenkins
```

## Creando Jenkins-Agent VM

- Crear una VM DS1_v2 con 1CPUs y 3.5GiB de memoria RAM y 15GB o más de almacenamiento.

```
sudo chmod 700 Jenkins-Agent_key.pem
```

```
ssh -i <Key-pair .pem> azureuser@<Public IP>
```

```
sudo init 6
```

```
sudo apt install openjdk-17-jre-headless
```

```
sudo apt-get install docker.io
```

```
sudo usermod -aG docker $USER
```

```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

```

```
sudo init 6
```

## Creando SonarQube VM

- Crear una VM B2s con 2CPUs y 4GiB de memoria RAM y 15GB o más de almacenamiento.

- Habilitarle el ppuerto TCP 9000.

- Instalar y configurar una base de datos postgresql:

```
sudo apt update
sudo apt upgrade
```

Para continuar y acceder al repositorio completo debes seguirme en mi cuenta de GitHub y solicitar acceso mediante cualquiera de los siguientes canales de comunicación:

<div align="center">

[![GitHub IgnaLog](https://img.shields.io/github/followers/IgnaLog?label=follow&style=social)](https://github.com/IgnaLog)

[![Linkedin: IgnacioLlorente](https://img.shields.io/badge/-IgnacioLlorente-blue?style=flatsquare&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/igna-llorente/)](https://www.linkedin.com/in/igna-llorente/)
[![Gmail IgnaLog](https://img.shields.io/badge/Gmail-ignacio.coding%40gmail.com-success)](mailto:ignacio.coding@gmail.com)
