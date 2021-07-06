
# Crear Imagen Ubuntu con Kubectl

En este laboratorio, descargará una imagen de Ubuntu 20.04 y sobre ésta instalará las diferentes herramientas necesarias para desplegar las aplicaciones hacia Kubernetes "**OnPremise**" o "**InCloud**".

Luego configurará la imagen como un runner de GitLab para integrarlo dentro de sus pipelines CI/CD.

## Paso 1: Descargar Imagen Ubuntu

    docker run -it -d --name kubectl-lnx ubuntu

## Paso 2: Conectarse al Contenedor

    docker exec -it kubectl-lnx bash

## Paso 3: Actualizar Paquetes

    apt-get update

## Paso 4: Instalar Curl

Para descargar los diferentes paquetes en Ubuntu debe instalar *curl*

    apt-get install curl

## Paso 5: Instalar Azure CLI

Si desea interactuar con Azure para acceder a algun recurso como ***Azure Kubernetes Service***, debe instalar *Azure CLI* con la siguiente instrucción:

    curl -sL https://aka.ms/InstallAzureCLIDeb | bash

## Paso 6: Instalar Kubectl

Para acceder al cluster de kubernetes, debe instalar ***Kubectl CLI*** por medio de las siguientes instrucciones:

    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https:/storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

Asigne permisos de ejecución sobre el archivo descargado.

    chmod +x ./kubectl

Cambie la ubicación del archivo para poderlo ejecutar sobre el contexto del shell sin inconvenientes:

    mv ./kubectl /usr/local/bin/kubectl

Verifique la version instalada:

    kubectl version --client

## Paso 7: Instalar Helm

Para integrar el administrador de paquetes Helm con su Cluster de Kubernetes, debe realizar la instalación del CLI con las siguientes instrucciones:

    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

Asigne permisos de ejecución sobre el archivo descargado.

    chmod 700 get_helm.sh

Ejecute el siguiente script para hacer la instalación:

    ./get_helm.sh

## Paso 8: Azure Login

Para conectarse al Cluster dentro de Azure, debe iniciar sesion para descargar las credenciales de acceso:

    az login

## Paso 9: Obtener Credenciales AKS

    az aks get-credentials -g tivit -n qcAks --overwrite-existing

## Paso 10: Instalar GIT

Para poder ejecutar exitosamente los pipelines de GitLab dentro de este contenedor, debe instalar Git para descargar el contenido del repositorio.

    apt install git

## Paso 11: Instalar GitLab Runner

Si desea utilizar este contenedor como un runner, siga las instrucciones del panel de configuración **Settings > CI/CD > Runners** para configurar el runner para **linux**.

Luego, podra referenciarlo dentro de su pipeline junto con los script de kubernetes. aqui tienes un archivo YAML de ejemplo:

    # Deployment step
    deploy:
      stage: deploy
      script:
        - kubectl version --client
        - kubectl config view   
        - mkdir -p $HOME/.kube
        - mv config $HOME/.kube/config
    
        - kubectl config get-contexts
        - kubectl config view
    
        - kubectl get all
        - kubectl apply -f azure-vote.yaml
      tags:
        - Kube

## Buen Trabajo
