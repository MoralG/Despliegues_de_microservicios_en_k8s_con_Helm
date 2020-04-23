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

Una vez agregado, podemos listar los chart con el siguiente comando:

~~~
helm search repo stable
~~~

Nos saldrá una lista de chart oficiales para poder instalar en nuestro cluster

~~~
NAME                                 	CHART VERSION	APP VERSION            	DESCRIPTION                                       
stable/acs-engine-autoscaler         	2.2.2        	2.1.1                  	DEPRECATED Scales worker nodes within agent pools 
stable/aerospike                     	0.3.2        	v4.5.0.5               	A Helm chart for Aerospike in Kubernetes          
stable/airflow                       	6.7.1        	1.10.4                 	Airflow is a platform to programmatically autho...
stable/ambassador                    	5.3.1        	0.86.1                 	A Helm chart for Datawire Ambassador              
.
.
.
~~~

### Instalando un Chart oficial

Vamos a instalar un chart del repositorio oficial de Helm, para hacer esto tenemos actializar primero la información de los chart disponibles localmentem para descargar si ha habido cambios.

~~~
helm repo update
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "stable" chart repository
    Update Complete. ⎈ Happy Helming!⎈ 
~~~

No ha salido un mensaje de que esta todo actuaizado, ahora vamos a proceder a instalar un chart. En nuestro caso vamos a probar con MySQL.

Antes de empezar a instalar, tenenemos que saber el nombre del chart, para eso utilizamos el comando `helm search repo`.

-------------------------------------------
#### Estructura:
* **`helm search repo [keyword] [flags]`**

###### [Para saber más sobre los comandos de helm]() o utilice `helm help` para una descripción general o utilice el parámetro `-h` para una descripción de un comando concreto, ejemplo `helm install -h`
-------------------------------------------

Vamos a buscar la **release stable** de mysql, pero si queremos otras versiones podemos utilizar la flags `--devel` para prerelease o `--version [version]` para que nos muestre la version concreta.

~~~
helm search repo mysql
    NAME                            	CHART VERSION	APP VERSION	    DESCRIPTION                                       
    stable/mysql                    	1.6.3        	5.7.28     	Fast, reliable, scalable, and easy to   use open-...
    stable/mysqldump                	2.6.0        	2.4.1      	A Helm chart to help backup MySQL   databases usi...
    stable/prometheus-mysql-exporter	0.5.2        	v0.11.0    	A Helm chart for prometheus mysql   exporter with...
    stable/percona                  	1.2.1        	5.7.26     	free, fully compatible, enhanced, open  source d...
    stable/percona-xtradb-cluster   	1.0.3        	5.7.19     	free, fully compatible, enhanced, open  source d...
    stable/phpmyadmin               	4.3.5        	5.0.1      	DEPRECATED phpMyAdmin is an mysql   administratio...
    stable/gcloud-sqlproxy          	0.6.1        	1.11       	DEPRECATED Google Cloud SQL     Proxy                 
    stable/mariadb                  	7.3.14       	10.3.22    	DEPRECATED Fast, reliable, scalable,    and easy t...
~~~

Nos ha listado todo lo que tiene que ver con el término **mysql** en su versión estable. Ahora vamos a instalar el chart **stable/mysql** con la versión 1.6.3 del chart.

Para instalar un chart tenemos que utilizar el comando `helm install`.

-------------------------------------------
##### Estructura:
* **`helm install [NAME] [CHART] [flags]`**

###### [Para saber más sobre los comandos de helm]() o utilice `helm help` para una descripción general o utilice el parámetro `-h` para una descripción de un comando concreto, ejemplo `helm install -h`
-------------------------------------------

Vamos a indicarle un el nombre de **maria** a nuestro chart, pero podemos utilizar la flags `--generate-name`para asignarle uno automáticamente.

~~~
helm install maria stable/mysql
~~~

Al instalar dicho chart,en el caso de mysql, nos sale una información del chart, como se muestra a continuación: 

~~~
NAME: maria
LAST DEPLOYED: Thu Apr 23 17:09:13 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
maria-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default maria-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h maria-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/maria-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
~~~

Ya tenemos instalado nuestro chart y nos muestra varia información, como la descripción, el nombre, el namespace, etc. Pero además nos muestra la instrucciones para conectarse a la base de datos. 

Para ver los chart que estan lanzados con Helm podemos utilizar el comando `helm ls`.


~~~
helm ls
    NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP     VERSION
    maria	default  	1       	2020-04-23 17:39:05.877933281 +0000 UTC	deployed	mysql-1.6.3	5.7.28  
~~~

### Desinstalando una versión

Para desinstalar una versión tenenemos que utilizar el comando `helm uninstall`.

-------------------------------------------
##### Estructura:
* **`helm uninstall RELEASE_NAME [...] [flags]`**

###### [Para saber más sobre los comandos de helm]() o utilice `helm help` para una descripción general o utilice el parámetro `-h` para una descripción de un comando concreto, ejemplo `helm install -h`
-------------------------------------------

Vamos a realizar la desinstalación del chart **maria** pero vamos a añadir el flag `--feep-history` para mantener el historial de versión.

~~~
helm uninstall maria --keep-history
    release "maria" uninstalled
~~~

Con este flag lo que conseguimos es poder rastrear las versiones incluso después de haberlas desinstalado, podiendo auditar el historial de un cluster e incluso recuperar una versión con el comando `helm rollback`.

Podemos ver con el comando `helm status`, el estado actual del chart.

~~~
helm status maria
NAME: maria
LAST DEPLOYED: Thu Apr 23 17:39:05 2020
NAMESPACE: default
STATUS: uninstalled
.
.
.
~~~

Nos muestra que ha sido desinstalada.

### Realizar un Rollback

Vamos a realizar un rollback para revertir una release a una versión anterior con el comando `helm rollback`.

-------------------------------------------
##### Estructura:
* **`helm rollback <RELEASE> [REVISION] [flags]`**

###### [Para saber más sobre los comandos de helm]() o utilice `helm help` para una descripción general o utilice el parámetro `-h` para una descripción de un comando concreto, ejemplo `helm install -h`
-------------------------------------------

Si hemos borrado una versión de un chart y por consiguiente, no nos sale con el comando `helm ls` vamos a tener que utilizar `helm history`.

~~~
helm history maria
    REVISION	UPDATED                 	STATUS     	CHART      	APP VERSION	DESCRIPTION            
    1       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	5.7.28     	Uninstallation complete
~~~

Sabiendo esto vamos a realizar el **rollback**.

~~~
helm rollback maria 1
    Rollback was a success! Happy Helming!
~~~

Nos ha indicado que esta todo correcto y listamos de nuemo con `helm ls` los chart lanzados:

~~~
helm ls
    NAME 	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP     VERSION
    maria	default  	2       	2020-04-23 17:51:41.84658262 +0000 UTC	deployed	mysql-1.6.3	5.7.    28     
~~~

Como podemos ver, el chart **maria** aparece de nuevo pero con una diferencia, la revisión es la número 2.

También podemos ver con `helm history` que nos aparece la revisión 1 desinstalada y la revisión 2 lanzada. 

~~~
helm history maria
    REVISION	UPDATED                 	STATUS     	CHART      	APP VERSION	DESCRIPTION            
    1       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	5.7.28     	Uninstallation complete
    2       	Thu Apr 23 17:51:41 2020	deployed	mysql-1.6.3	5.7.28     	Uninstallation complete
~~~