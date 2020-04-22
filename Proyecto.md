# Despliegues de microservicios en k8s con Helm (y OpenShift)

### Introducción

Helm es un administrador de paquetes para Kubernetes, que ayuda en el proceso de gestión de versiones a desplegar, su empaquetado, proceso de release (forward, rollback, upgrade), etc... de una manera más fácil y rápida.

Estos paquetes se denominan chart, los cuales son una colección de ficheros que describen un conjunto de recursos del API de Kubernetes. Un ejemplo comparable para entenderlo sería el caso del `apt` o `yum` o otros administradores de paquetes de distribuciones de linux pero para Kubernetes.

Helm es un proyecto oficial de Kubernetes y se esta empezado a utilizar mucho por la facilidad de los usuarios a la hora de trabajar con los paquetes de Kubernetes.

Las claves de la utilización de Helm son:

* Utilización de chart para la empaquetación de nuestras aplicaciones.
* Insteractuar con repositorios de chart, ya sean públicos o privados.
* Instalar y desinstalar chart en nuestros cluster de Kubernetes.
* Instalar automáticamente dependencias de software.
* Gestionar el ciclo de vida de despliegue de chart que han sido instaladas con Helm.

Para realizar todas estan funcionalidades, Helm utiliza los siguientes componentes:

* **Helm**, una herramienta de line de comandos que proporciona al usuario todas las funcionalidades de Helm.
* **Tiller**, Un componente de servidor complementario que atiende los comandos de helm desde el cluster de Kubernetes y maneja la configuración y la implementación de las versiones de software en el cluster.


### Requisitos previos

* Un clúster de Kubernetes en la versión 1.8 o posterior, con el control de acceso en roles (RBAC) hablitado.

###### [Instalación Kubernetes con Kubeadm]()

###### Comprobar la versión de Kubernetes:
~~~
kubectl version
    Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
    Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
~~~

* Tener la herramienta de líneas de comando `kubectl` instalada en su equipo local, configurada para poder conectarse al clúster.

###### Comprobar la conectividad:
~~~
kubectl cluster-info
    Kubernetes master is running at https://10.0.0.10:6443
    KubeDNS is running at https://10.0.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
~~~

### Instalación de Helm

Vamos a instala la herramienta de linea de comando `helm` en nuestro equipo local de trabajo.

Descargamos la última release de Helm de la versión 3. Para hacer esto nos vamos a la [página oficial](https://github.com/helm/helm/releases) y nos descargamos el fichero `.tar.gz` para Linux

> **NOTA**: Es recomendable que nos descarguemos una versión que este **Verificada**.

En el momento de la creación de este tutorial, nos descargamos la versión 3.2.0 de Helm.
~~~
wget https://get.helm.sh/helm-v3.2.0-linux-amd64.tar.gz
~~~

Descromprimimos el fichero:
~~~
tar -zxvf helm-v3.2.0-linux-amd64.tar.gz 
~~~

Tenemos que mover el binario del directorio que hemos desempaquetado a la dirección, en mi caso, `/usr/local/bin/helm` 

~~~
sudo mv linux-amd64/helm /usr/local/bin/helm
~~~

Comprobamos la versión
~~~
helm version
    version.BuildInfo{Version:"v3.2.0", GitCommit:"e11b7ce3b12db2941e90399e874513fbd24bcb71",   GitTreeState:"clean", GoVersion:"go1.13.10"}
~~~

Ya tenemos instalado Helm en la version 3, lo siguiente que vamos a ver es como iniciar el repositorio chart de Helm.

### Agregando un Helm Chart Repository

Vamos a agregar el repositorio de chart de Helm para poder instalar los chart que queramos.

~~~
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
    "stable" has been added to your repositories
~~~

Una vez agregado podemos listar los chart que podemos instalar con el siguiente comando:

~~~
helm search repo stable
~~~

Nos saldrá una lista de repositorios estables.

