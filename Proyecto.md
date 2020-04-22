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
sudo kubectl version
    Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0",   GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean",   BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
    Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0",   GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean",   BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
~~~

* Tener la herramienta de líneas de comando `kubectl` instalada en su equipo local, configurada para poder conectarse al clúster.

###### Comprobar la conectividad:
~~~
sudo kubectl cluster-info
    Kubernetes master is running at https://10.0.0.10:6443
    KubeDNS is running at https://10.0.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
~~~


### Instalación de Helm

Vamos a instala la herramienta de linea de comando `helm` en nuestro equipo local de trabajo.

Tenemos que descargarnos un script con comandos, proporcionado oficialmente en el repositorio de GitHub de Helm.

~~~
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
~~~

Cambiamos los permisos del script para que sea ejecutable

~~~
chmod u+x install-helm.sh
~~~

Para ejecutar el script utilizamos el siguiente comando:

~~~
./install-helm.sh
~~~

Nos tendrá que salir un mensaje al finalizar como el siguiente:

~~~
helm installed into /usr/local/bin/helm
Run 'helm init' to configure helm.
~~~

Y estará la herramienta Helm instalada.

### Instalación de Tiller

Tiller es un complemento del comando helm que se ejecuta en su clúster, 