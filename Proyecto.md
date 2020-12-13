# Despliegues de microservicios en k8s con Helm

## Introdución

Helm es un administrador de paquetes para Kubernetes que ayuda en el proceso de gestión de versiones a desplegar, empaquetar, proceso de release (forward, rollback, upgrade), etc. de una manera fácil y rápida.

A través de una colección de ficheros, los denominados chart, se describen los recursos de la API de Kubernetes. Un ejemplo comparable sería el caso de `apt`, `yum` u otros administradores de paquetes de distribuciones Linux pero para Kubernetes.


Las claves de la utilización de Helm son:

* Instalar automáticamente dependencias de software.
* Utilización de chart para la empaquetación de nuestras aplicaciones.
* Insteractuar con repositorios de chart, ya sean públicos o privados.
* Los Helm Charts sirven para describir incluso las aplicaciones más complejas. Ofrecen una instalación repetible de la aplicación, manteniendo un único punto de control.
* El proceso de “rollback” con Helm Charts es sencillo.
* Las actualizaciones de Helm Charts son sencillas y más fáciles de utilizar para los desarrolladores.
* Gestionar el ciclo de vida de despliegue de chart que han sido instaladas con Helm.

Helm es un proyecto oficial de Kubernetes y se esta empezado a utilizar mucho por la facilidad de los usuarios a la hora de trabajar con los paquetes de Kubernetes. Es mantenido por la CNCF en colaboración con Microsoft, Google, Bitnami y la comunidad de Helm.

En resumen, la principal función de Helm es definir, instalar y actualizar aplicaciones complejas de Kubernetes.

#### Requisitos previos

-------------------

* Un clúster de Kubernetes en la versión 1.8 o posterior, con el control de acceso en roles (RBAC) hablitado. Si necesita ayuda para instalar Kubernetes con kubeadm [LEER AQUÍ](https://github.com/MoralG/Trabajando_con_Kubernetes/blob/master/Trabajando_con_Kubernetes.md)

###### Comprobar la versión de Kubernetes:
***debian@cliente:**~* **$** `kubectl version`
~~~
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc"Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc"Platform:"linux/amd64"}
~~~

--------------------------------------

* Tener la herramienta de líneas de comando `kubectl` instalada en su equipo local, configurada para poder conectarse al clúster.

###### Comprobar la conectividad:
***debian@cliente:**~* **$** `kubectl cluster-info`
~~~
Kubernetes master is running at https://10.0.0.10:6443
KubeDNS is running at https://10.0.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
~~~

----------------

## Instalación de Helm

Se va a instalar la herramienta de línea de comandos `helm` en el equipo local de trabajo.

Se descarga la última release de Helm desde la [página oficial](https://github.com/helm/helm/releases) que será un fichero `.tar.gz` para Linux. 

> #### NOTA 
> -----------------------
>
> Es recomendable descargarse una versión **Verificada**.

En este caso, la última versión y con la que se va a trabajar será Helm 3.2.0.

***debian@cliente:**~/tmp* **$** `wget https://get.helm.sh/helm-v3.3.4-linux-amd64.tar.gz`

Se descomprime el fichero:

***debian@cliente:**~/tmp* **$** `tar -zxvf helm-v3.3.4-linux-amd64.tar.gz`

Hay que mover el binario del directorio que se ha desempaquedo a `/usr/local/bin/helm`:

***debian@cliente:**~/tmp* **$** `sudo mv linux-amd64/helm /usr/local/bin/helm`

Se comprueba la versión:

***debian@cliente:**~* **$** `helm version`
~~~
version.BuildInfo{Version:"v3.2.0", GitCommit:"e11b7ce3b12db2941e90399e874513fbd24bcb71",   GitTreeState:"clean", GoVersion:"go1.13.10"}
~~~


> ##### NOTA
> --------------------
> Para mayor seguridad es recomendable quitar los permisos de lectura a los grupos para el fichero de configuración del cluster.
>
> Ejecutando el siguiente comando:
> ***debian@cliente:**~* **$** `sudo chmod 600 /home/debian/.kube/mycluster.conf`


Una vez instalado Helm 3, lo siguiente es indicar el repositorio chart de Helm.

### Agregar un Helm Chart Repository

Se agrega el repositorio de chart de Helm para poder instalar los chart que sean necesarios.

***debian@cliente:**~* **$** `helm repo add stable https://kubernetes-charts.storage.googleapis.com/`
~~~
"stable" has been added to your repositories
~~~

Con el repositorio agregado se pueden listar los chart con el siguiente comando:

***debian@cliente:**~* **$** `helm search repo stable`
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

## Guía rápida

### Instalación de un Chart oficial

A continuación, se va a instalar un chart del repositorio oficial de Helm. Para ellos, en primer lugar, hay que actualizar la información de los chart disponibles en el repositorio `stable` con el comando `helm repo update`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm repo update [flags]`**
>
> ###### Más información sobre el comando repo update en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_repo_update/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

***debian@cliente:**~* **$** `helm repo update`
~~~
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
~~~

Tras completar la actualización, se procede a la instalación de un chart, en este caso MySQL.

Antes de empezar a instalar, hay que saber el nombre del chart. Para eso se utiliza el comando `helm search repo`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm search repo [keyword] [flags]`**
> 
> ###### Más información sobre el comando search repo en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_search_repo/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

Se busca la **release stable** de mysql. Para buscar otras versiones se utiliza utiliza el parámetro `--devel` para prerelease o `--version [version]` para versiones concretas.

***debian@cliente:**~* **$** `helm search repo mysql
~~~
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

De esta forma, lista todo lo que tiene que ver con el término **mysql** en su versión estable. En este caso, se va a instalar el chart **stable/mysql** con la versión 1.6.3 del chart.

Para instalar un chart hay que utilizar el comando `helm install`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm install [NAME] [CHART] [flags]`**
> 
> ###### Más información sobre el comando install en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_install/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

Se le asigna el nombre de **maria** al chart. También se puede asignar un nombre automáticamente con el parámetro `--generate-name`.

***debian@cliente:**~* **$** `helm install maria stable/mysql`

Al instalar dicho chart, en el caso de mysql, aparece la información del chart, como se muestra a continuación:
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

> #### NOTA
> --------------------
> Para ver la opciones configurables de un chart: `helm show values <nombre_chart>`
> 
> ***debian@cliente:**~* **$** `helm show values stable/mariadb`
> ~~~
> ## Global Docker image parameters
> ## Please, note that this will override the image parameters, including dependencies, configured to use > the global value
> ## Current available global Docker image parameters: imageRegistry and imagePullSecrets
> ##
> # global:
> #   imageRegistry: myRegistryName
> #   imagePullSecrets:
> #     - myRegistryKeySecretName
> #   storageClass: myStorageClass
>
> ## Use an alternate scheduler, e.g. "stork".
> ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
> ##
> # schedulerName:
>
> ## Bitnami MariaDB image
> ## ref: https://hub.docker.com/r/bitnami/mariadb/tags/
> ##
> image:
>   registry: docker.io
>   repository: bitnami/mariadb
>   tag: 10.3.22-debian-10-r27
> .
> .
> .
> ~~~
> Las opciones se pueden modificar de dos formas:
> * Indicando los parámetros en un fichero `.yaml`:
>
> Se crea el fichero `.yaml` indicando las opciones:
> 
> ***debian@cliente:**~* **$** `echo '{mariadbUser: usuario1, mariadbDatabase: usuario_bd}' > prueba.yaml`
> 
> Y se referencia dicho fichero `.yaml` en la instalación, con el parámetro `-f`:
> 
> ***debian@cliente:**~* **$** `helm install -f prueba.yaml stable/mariadb --generate-name`
>
> * Indicando las opciones con el parametro `--set` en la instalación:
> 
> ***debian@cliente:**~* **$** `helm install stable/mariadb --generate-name --set name=usuario1`
> 

Una vez instalado el chart de MySQL, se muestra información como la descripción, el nombre, el namespace, etc. Además, muestra las instrucciones para conectarse a la base de datos.


Para ver los chart lanzados con Helm se utiliza el comando `helm ls`.

***debian@cliente:**~* **$** `helm ls`
~~~
NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP     VERSION
maria	default  	1       	2020-04-23 17:39:05.877933281 +0000 UTC	deployed	mysql-1.6.3	5.7.28
~~~

### Actualización de una versión

Para lanzar una nueva versión de un chart se utiliza el comando `helm upgrade`.

Una actualización toma una versión existente y la actualiza de acuerdo con la información que se proporcione. En el caso de Helm solo se actualizan las cosas que han cambiado desde la última versión.

Como ejemplo, se va a cambiar la configuración del chart instalado anteriormente. Creando un fichero `yaml` con la opción `mariadbUser` modificada y se crea una nueva versión.

***debian@cliente:**~* **$** `echo '{mariadbUser: usuario1}' > prueba.yaml`

***debian@cliente:**~* **$** `helm upgrade -f prueba.yaml maria stable/mariadb`

***debian@cliente:**~* **$** `helm get values maria`
~~~
USER-SUPPLIED VALUES:
mariadbUser: usuario1
~~~


***debian@cliente:**~* **$** `helm history maria`
~~~
REVISION	UPDATED                 	STATUS     	CHART         	APP VERSION	DESCRIPTION
1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
~~~

### Desinstalación de una versión

Para desinstalar una versión hay que utilizar el comando `helm uninstall`.


> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm uninstall RELEASE_NAME [...] [flags]`**
> 
> ###### Más información sobre el comando uninstall en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_plugin_uninstall/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

Se va a desinstalar el chart **maria** con la opción `--feep-history` para mantener el historial de versión.

***debian@cliente:**~* **$** `helm uninstall maria --keep-history`
~~~
release "maria" uninstalled
~~~

Con esta opción se consigue un rastreamiento de las veriones incluso después de la desinstalación, pudiendo auditar el historial de un cluster e incluso recuperar una versión con el comando `helm rollback`.

Con el comando `helm status` se ve el estado actual del chart. En este caso, ha sido desinstalado:


***debian@cliente:**~* **$** `helm status maria`
~~~
NAME: maria
LAST DEPLOYED: Thu Apr 23 17:39:05 2020
NAMESPACE: default
STATUS: uninstalled
.
.
.
~~~

### Rollback

Se va a realizar un rollback para revertir una release a una versión anterior con el comando `helm rollback`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm rollback <RELEASE> [REVISION] [flags]`**
> 
> ###### Más información sobre el comando rollback en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_rollback/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

Si se ha borrado una versión de un chart y, por consiguiente, no aparece con `helm ls` hay que utilizar `helm history`.

***debian@cliente:**~* **$** `helm history maria`
~~~
REVISION	UPDATED                 	STATUS     	CHART      	APP VERSION	DESCRIPTION
1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
2       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	    5.7.28     	Uninstallation complete
~~~

Conociendo la revisión a la que se quiere revertir se realiza el **rollback**:

***debian@cliente:**~* **$** `helm rollback maria 1`
~~~
Rollback was a success! Happy Helming!
~~~

A continuación, se listan los chart para comprobar que se ha realizado correctamente:

***debian@cliente:**~* **$** `helm ls`
~~~
NAME 	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART      	APP     VERSION
maria	default  	2       	2020-04-23 17:51:41.84658262 +0000 UTC	deployed	mysql-1.6.3	5.7.    28
~~~

El chart **maria** aparece de nuevo pero con la revisión número 2.

Ahora, con `helm history` aparece la revisión 1 desinstalada y la revisión 2 lanzada.

***debian@cliente:**~* **$** `helm history maria`
~~~
REVISION	UPDATED                 	STATUS     	CHART      	APP VERSION	DESCRIPTION
1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
2       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	    5.7.28     	Uninstallation complete
3       	Thu Apr 23 17:51:41 2020	deployed	mysql-1.6.3	    5.7.28     	Uninstallation complete
~~~

## Creación de Charts en Helm

A continuación, se van a desarrollar Charts propios en Helm.

Como se dijo anteriormente, un chart es una colección de ficheros que describen un cojunto de recursos de Kubernetes. Se puede usar un solo chart para implementar un pod memcached, o, si es algo más complicado, una pila completa de aplicaciones web con servidores HTTP, bases de datos, cache, etc.

Se va a crear un chart paso a paso para una aplicación CRUD (Create, Read, Update, Delete) en Java. Después, se estudiará cómo sacarle partido a los chart oficiales o a los creados por la comunidad, desplegando un chart oficial `stable/lamp` pero añadiendo valores propios y secrets.

### Despliegue de Chart CRUD utilizando express.js y mongodb

Se va a desplegar una aplicación CRUD, aplicación simple donde crear, leer, actualizar y elilminar datos, con la ayuda de Express.js, uno de los framework de aplicaciones para Node.js. También será necesario una base de datos, en este caso se va a utilizar MongoDB, que se introducirá como dependencia.

Se crearan todos los objetos necesarios para el depliegue en Kubernetes con ficheros YAML pero modificándolos para que sean funcionales para utilizarlos con Helm. Además, se explicará para qué se utilizan las dependencias en los chart de Helm.

> #### NOTA
> ----------------------
> Helm trabaja con un lenguaje de plantillas Go (Lenguaje de programación desarrollado por Google) añadidos a los fichero YAML de los objetos de Kubernetes.
> 
> Ejemplo:
> ````yaml
> apiVersion: v1
> kind: ConfigMap
> metadata:
>   name: {{ .Release.Name }}-configmap
> data:
>   myvalue: "Hola Mundo"
>   drink: {{ quote .Values.favorite.drink }}
>   food: {{ quote .Values.favorite.food }}
> ````
> La estructura es muy fácil, se basa en el uso de cuatro llaves `{{ }}` dónde se indican las funciones y atributos.
> ~~~~
> functionName arg1 arg2 ...
> ~~~~
> Hay varias tipos de funciones que se pueden utilizar. Para saber más sobre los tipos de funciones [LEER AQUÍ](https://helm.sh/docs/chart_template_guide/function_list/)
> 
> Hay que tener claro los argumentos, ya que al utilizar `.Values`, el valor predeterminado que va a adquirir es el que se indica en el fichero `values.yaml`.
> 
> Ejemplo de fichero values.yaml:
> ~~~~
> favorite:
>   drink: coffee
>   food: pizza
> ~~~~

Creación del chart:
***debian@cliente:**~* **$** ``helm create app-crud``
~~~
Creating app-crud
~~~

Al ejecutar el comando anterior, se crean los directorios y ficheros base para el chart. Éstos se modificarán para configurar la aplicación y se desplegará, con una release concreta, y de forma creciente si se actualizara.

~~~
app-crud/
├── charts
|   └──
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 8 files
~~~

Los ficheros y directorios necesarios para relizar un buen despliegue en Kubernetes son:

|Objeto          | Descripción
|----------------|------------------------------
|**charts/**| Directorio donde se añadirán los chart necesarios como dependencias.
|**Chart.yaml**| Fichero YAML que contiene la información del chart.
|**README.md**| Fichero utilizado para la descripción del chart.
|**values.yaml**| Fichero que contiene los distintos valores por defectos (que le pasamos al chart).
|**templates/**| Directorio con los recursos de Kubernetes. Se añaden los diferentes objetos mediante ficheros YAML.
|**templates/NOTES.txt**| Fichero opcional, que contiene breves notas de uso. Estas se muestran al terminar el comando `helm install`.
|**templates/_helpers.tpl**| Fichero opcional, utilizado para definir algunos valores que pueden ser reutilizables en la creación del chart.

Por último, se elige la imagen o imágenes que se va a utilizar, las dependencias que se necesitan definir y la necesidad de volúmenes persistentes.

#### Fichero Chart.yaml

En primer lugar, se modifica los metadatos del fichero `Chart.yaml`.

```yaml
#Versión de la API del chart. Sube con un nuevo despliegue del chart.
apiVersion: v1
#Versión de la aplicación que se va a desplegar.
appVersion: "1.0.0"
description: Aplicacion CRUD en Helm chart con express.js y mongodb.
# Nombre del chart
name: app-crud
# Versión del empaquetado del chart.
version: 0.1.0
# Alojamiento del chart.
sources:
- https://github.com/MoralG/appCRUD
maintainers:
- name: moralg
  email: ale95mogra@gmail.com
icon: https://res.cloudinary.com/practicaldev/image/fetch/s--5IllY723--/c_imagga_scale,f_auto,fl_progressive,h_900,q_auto,w_1600/https://thepracticaldev.s3.amazonaws.com/i/a3exuz06e9h212pandfr.png
```

Se define la dependencia. Dado que la aplicación necesita una base de datos, MongoDB, se necesita especificarla en el campo `dependencies` del fichero `Chart.yaml`.

```yaml
dependencies:
# Nombre del chart.
- name: mongodb
# Versión del empaquetado del chart, en este caso la última estable "latest". 
 version: latest
# Alojamiento.
repository: https://kubernetes-charts.storage.googleapis.com/
# Opcional. Esto hará que el fichero "values.yaml" del chart principal se puedan añadir valores, mientras este el atributo "mongodb.enabled" en "true".
 condition: mongodb.enabled
```

Una vez estén las dependencias definidas hay que sincronizarlas con las dependencias reales almacenadas en el directorio `chart/`. Para ello, se utiliza el comando `helm dep update`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm dep update CHART [flags]`**
>
> ###### Más información sobre el comando dep update en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_dependency_update/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.

***debian@cliente:**~/app-crud* **$** `helm dep update`
```shell
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "django" chart repository
...Successfully got an update from the "bitnami" chart repository
...Unable to get an update from the "stable" chart repository (https://charts.helm.sh/stable):
        Get "https://charts.helm.sh/stable/index.yaml": dial tcp: lookup charts.helm.sh on 192.168.202.2:53: server misbehaving
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading mongodb from repo https://kubernetes-charts.storage.googleapis.com/
Deleting outdated charts
```

Como muestra en la salida del comando anterior, un chart se ha guardado. Se comprueba en el directorio `chart/`.

***debian@cliente:**~/app-crud* **$** `ls -l charts/`
~~~
-rw-r--r-- 1 debian debian 5742 Nov 30 09:27 mongodb-2.0.5.tgz
~~~

#### Fichero deployment.yaml

Lo siguiente será modificar el fichero `deployment.yaml`, que es el recurso que se va a encargar de definir el control de réplicas, escabilidad de pods, etc.

Se añaden algunas label para tener más control y comodidad al listar y trabajar con los objetos se van a crear.

```yaml
labels:
  app: {{ template "express-crud.name" . }}
  chart: {{ template "express-crud.chart" . }}
  release: {{ .Release.Name }}
  heritage: {{ .Release.Service }}
```

> #### NOTA
> ----------------------
> La función **template** hace referencia al código definido en el fichero `_helpers.tlp`. Se muestra a continuación:
>
> ~~~~
> {{- define "express-crud.name" -}}
> {{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
> {{- end -}}
> ~~~~
> Esto define el atributo `express-crud.name` con el nombre del chart extraido del fichero `Chart.yaml` y se puede unir una cadena extra si se añade el argumento `nameOverride` al fichero ``values.yaml``.
>
> ~~~~
> {{- define "express-crud.chart" -}}
> {{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" -}}
> {{- end -}}
> ~~~~
> Esto define el atributo `express-crud.chart` con el nombre unido a la version del chart extraido del fichero `Chart.yaml`.
>
> 
> Hay que tener en cuenta que: 
> 
> * `.Chart` hace referencia al fichero `Chart.yaml`.
> 
> * `.Values` hace referencia al fichero `values.yaml`.
> 
> * `.Release` hace referencia al nombre del objeto asignado en el despliegue.

Se añaden variables de entorno para configurar los valores del chart de MongoDB.

```yaml
env:
- name: DATABASE_USER
  value: '{{ .Values.mongodb.mongodbUsername }}'
- name: DATABASE_NAME
  value: '{{ .Values.mongodb.mongodbDatabase }}'
- name: DATABASE_HOST
  value: '{{ .Release.Name }}-mongodb'
- name: DATABASE_PORT
  value: '27017'
- name: DATABASE_PASSWORD
  valueFrom:
    secretKeyRef:
      name: {{ .Release.Name }}-mongodb
      key: mongodb-password
```

* Valores del fichero `values.yaml` referentes a la parte de `Values.mongodb`:
```yaml
mongodb:
  mongodbRootPassword:
  mongodbUsername: admin
  mongodbPassword:
  mongodbDatabase: test
```

Se modifica el parámetro `image` para actualizar el chart con Helm a una nueva versión de la aplicación cambiando el valor en `Chart.yaml`.

```yaml
image: "{{ .Values.image.repository }}:{{ default .Chart.AppVersion .Values.image.tag }}"
```

> #### NOTA
> ----------------------
> La función **default** se utiliza para asignar un valor por defecto en el caso que no se especifique el atributo en el fichero ``values.yaml``.
> 
> ``{{ default .Chart.AppVersion .Values.image.tag }}`` quiere decir que si no se especifica un atributo `image.tag` en el fichero `values.yaml`, se extraerá del atributo `AppVersion` del fichero `Chart.yaml`.

* Valores del fichero `values.yaml` referentes a la parte de `Values.image`:
```yaml
image:
  repository: jainishshah17/express-mongo-crud
  # tag: 1.0.1
  pullPolicy: IfNotPresent
```

Se añaden las siguientes líneas para poder modificar los valores referentes al puerto interno, protocolo, etc. desde el fichero `values.yaml`.

```yaml
ports:
- name: {{ .Values.service.name }}
  containerPort: {{ .Values.service.internalPort }}
  protocol: {{ .Values.service.protocol }}
```

Una buena práctica es agregar una prueba liveness y readiness para verificar el estado continuo de la aplicación. Para esto se utilizan las sondas.

```yaml
livenessProbe:
  httpGet:
    path: {{ .Values.livenessProbe.path }}
    port: {{ .Values.livenessProbe.port }}
  initialDelaySeconds: {{ .Values.livenessProbe.initialDelaySeconds }}
  periodSeconds: {{ .Values.livenessProbe.periodSeconds }}
  failureThreshold: {{ .Values.livenessProbe.failureThreshold }}
readinessProbe:
  httpGet:
    path: {{ .Values.readinessProbe.path }}
    port: {{ .Values.readinessProbe.port }}
  initialDelaySeconds: { .Values.readinessProbe.initialDelaySeconds }}
  periodSeconds: {{ .Values.readinessProbe.periodSeconds }}
  failureThreshold: {{ .Values.readinessProbe.failureThreshold }}
```

* Valores del fichero `values.yaml` referentes a la parte de `Values.livenessProbe` y `Values.readinessProbe`:
```yaml
livenessProbe:
  path: '/health'
  port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  failureThreshold: 10 
  
readinessProbe:
  path: '/health'
  port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  failureThreshold: 10 
```

Por último, se va a añadir un `initContainers` para mantener en pendiente el inicio de la aplicación hasta que la base de datos esté en funcionamiento.

```yaml
initContainers:
- name: wait-for-db
  image: "{{ .Values.initContainerImage }}"
  command:
  - 'sh'
  - '-c'
  - >
    until nc -z -w 2 {{ .Release.Name }}-mongodb 27017 && echo mongodb ok;
      do sleep 2;
    done
```

* Valores del fichero `values.yaml` referentes a la parte de `Values.initContainers`:
```yaml
initContainerImage: "alpine:3.6"
```

> #### NOTA
> ------------------------
> Lo que realiza el código añadido en `initContainers.command` es un bucle con una condición. La condición, `until`, es que mientras el contenedor `{{ .Release.Name }}-mongodb` no esté creado, el valor es falso y matiene el estado pendiente gracias al comando `sleep`, hasta que sale del bucle.

#### Fichero service.yaml

Con el fichero de despliegue listo, se va a modificar el fichero `service.yaml` para exponer nuestra aplicación al exterior.
Un servicio permite que la aplicación reciba trafico a través de una dirección IP. Los servicios se pueden exponer de diferentes formas especificando un tipo:

|Tipo            | Descripción
|----------------|-------------------------------
|**ClusterIP**| Solo se puede acceder al servicio mediante una IP interna desde el cluster.
|**NodePort**| Se puede acceder al servicio desde fuera del clúster a través de NodeIP y NodePort
|**LoadBalancer**| Se puede acceder al servicio desde fuera del clúster a través de un equilibrador de carga externo. Puede ingresar a la aplicación.


En este caso, se va a utilizar LoadBalancer para la aplicación CRUD, ya que se necesita acceder desde el exterior, y el servicio de mongodb será ClusterIP porque éste solo tiene que poder tener acceso a la aplicación CRUD dentro del cluster.

Se modifica el fichero `service.yaml` para indicarle los valores de tipo de servicio, puerto externo, etc.

```yaml
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.externalPort }}
      targetPort: http
      protocol: {{ .Values.service.protocol }}
      name: {{ .Values.service.name }}
```


* Valores del fichero `values.yaml` referentes a la parte de `Values.service`:
```yaml
service:
  name: http
  type: LoadBalancer
  internalPort: 3000
  externalPort: 80
  protocol: TCP
```

Se añadimen algunas label, para tener más control y comodidad al listar y trabajar con los objetos que se van a crear.

```yaml
labels:
  app: {{ template "express-crud.name" . }}
  chart: {{ template "express-crud.chart" . }}
  release: {{ .Release.Name }}
  heritage: {{ .Release.Service }}
```

#### Fichero values.yaml

Una buena práctica es definir la mayoría de la configuración en el fichero `values.yaml` para ayudar a mantener los chart Helm en un buen estado de mantenimiento, además de ser más fácil si se cambia a otra configuración.

El contenido del fichero `values.yaml` quedaría de la siguiente forma:
```yaml
# Default values for app-crud.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

#------------------------------------------------

initContainerImage: "alpine:3.6"
imagePullSecrets:
replicaCount: 1

#------------------------------------------------

## Configuración de los valores referentes a la imagen de la aplicación CRUD

image:
  repository: jainishshah17/express-mongo-crud
# Puede descomentar el atributo "tag" y tendrá un peso mayor que el valor por defecto.
  # tag: 1.0.1
  pullPolicy: IfNotPresent

#------------------------------------------------

## Configuración de la dependencia de mongodb
## ref: https://github.com/kubernetes/charts/blob/master/stable/mongodb/README.md
##
mongodb:
  enabled: true
  image:
    tag: 3.6.3
    pullPolicy: IfNotPresent
  persistence:
    size: 50Gi
  mongodbExtraFlags:
  - "--wiredTigerCacheSizeGB=1"
  mongodbRootPassword:
  mongodbUsername: admin
  mongodbPassword:
  mongodbDatabase: test

#------------------------------------------------

## Configuración de los valores de Readiness and Liveness Probes (deployment.yaml)

livenessProbe:
  path: '/health'
  port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  failureThreshold: 10 
  
readinessProbe:
  path: '/health'
  port: http
  initialDelaySeconds: 60
  periodSeconds: 10
  failureThreshold: 10 

#------------------------------------------------

# Configuración de valores referente a servicios (service.yaml, deployment.yaml)

service:
  name: http
  type: LoadBalancer
  internalPort: 3000
  externalPort: 80
  protocol: TCP

#------------------------------------------------

ingress:
  enabled: false

resources: {}

nodeSelector: {}

tolerations: []

affinity: {}
```

Con todo configurado, solo falta instalar el chart. Pero antes se examina éste para detectar posibles problemas con el comando `helm lint`.

> #### ESTRUCTURA DEL COMANDO
> --------------------
> * **`helm lint PATH [flags]`**
> 
>> ###### [Para saber más sobre los comandos de helm](https://helm.sh/docs/helm/helm_lint/) o utilice `helm help` para una descripción general o utilice el parámetro `-h` para una descripción de un comando concreto

> ###### Más información sobre el comando lint en la [página oficial de Helm]](https://helm.sh/docs/helm/helm_lint/) o con el comando `helm help` para una descripción general y con el parámetro `-h` para una descripción del comando concreto.


***debian@cliente:**~/app-crud* **$** `helm lint ./`
```shell
==> Linting ./

1 chart(s) linted, 0 chart(s) failed
```

Cuando se muestre el mensaje que indica que hat cero fallos se puede iniciar la instalación.

Es posible que algunos fallos estén relacionados con la `apiVersion`, ya que dependiendo de la version de helm las versiones de las dependencias pueden varias.

> #### NOTA
> --------------------------
> Ejemplo de error:
> ~~~
> Chart.yaml: dependencies are not valid in the Chart file with apiVersion 'v1'. They are valid in apiVersion 'v2'`
> ~~~
>
> Para arreglar este error hay que cambiar la versión del campo `apiVersion` a `v2` en el dichero `Chart.yaml`.

> #### NOTA
> --------------------------

> Para la dependencia, base de datos MongoDB, es necesario crea un Persistent Volume para guardar los datos. Es por eso que en uno de los ficheros `.yaml` crea un Persistem Volume Claim. A continuación, un ejemplo del fichero de creación de un PV:  
> ```yaml
> apiVersion: v1
> kind: PersistentVolume
> metadata:
>   name: task-pv-volume
>   labels:
>     type: local
> spec:
>   storageClassName: manual
>   capacity:
>     storage: 10Gi
>   accessModes:
>     - ReadWriteOnce
>   hostPath:
>     path: "/mnt/data"
> ```

Por último, se va a realizar la instalación del chart, con el mismo comando que se usó en la guía rápida, `helm install`. Este comando se encarga de desplegar todos los ficheros `.yaml` de los objetos de Kubernetes inyectándole los valores que se han indicado en el dichero `values.yaml`.

***debian@cliente:**~/app-crud* **$** `helm install app-crud ./`
~~~
NAME: app-crud
LAST DEPLOYED: Mon Nov 30 12:50:21 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get svc -w app-crud-express-crud'
  export SERVICE_IP=$(kubectl get svc --namespace default app-crud-express-crud -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP:80
~~~

> #### NOTA
> --------------------
> El mensaje que muestra se puede modificar editando el fichero `template/NOTES.txt`. Un ejemplo del contenido que se ha usado en esta configuración:
> ~~~~
> 1. Get the application URL by running these commands:
> {{- if .Values.ingress.enabled }}
> {{- range .Values.ingress.hosts }}
>   http{{ if $.Values.ingress.tls }}s{{ end }}://{{ . }}{{ $.Values.ingress.path }}
> {{- end }}
> {{- else if contains "NodePort" .Values.service.type }}
>   export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ template "express-crud.fullname" . }})
>   export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
>   echo http://$NODE_IP:$NODE_PORT
> {{- else if contains "LoadBalancer" .Values.service.type }}
>      NOTE: It may take a few minutes for the LoadBalancer IP to be available.
>            You can watch the status of by running 'kubectl get svc -w {{ template "express-crud.fullname" . }}'
>   export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ template "express-crud.fullname" . }} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
>   echo http://$SERVICE_IP:{{ .Values.service.externalPort }}
> {{- else if contains "ClusterIP" .Values.service.type }}
>   export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app={{ template "express-crud.name" . }},release={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
>   echo "Visit http://127.0.0.1:{{ .Values.service.externalPort }} to use your application"
>   kubectl port-forward $POD_NAME {{ .Values.service.externalPort }}:80
> {{- end }}
> ~~~~

La instalación se ha realizado con éxito y se puede comprobar con `helm list`.

***debian@cliente:**~/app-crud* **$** `helm list`
~~~
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                APP VERSION
app-crud        default         1               2020-11-30 12:50:21.429099337 +0000 UTC deployed        express-crud-0.1.0   1.0.1
~~~

Para ver los objetos que se han creado se ejecuta `kubectl get all`:

***debian@cliente:**~/app-crud* **$** `kubectl get all`
~~~
NAME                                         READY   STATUS    RESTARTS   AGE
pod/app-crud-express-crud-565b97d46d-knwkh   1/1     Running   0          5m12s
pod/app-crud-mongodb-5cc7c8c7d7-pfpdn        1/1     Running   0          5m12s

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/app-crud-express-crud   LoadBalancer   10.109.130.120   <pending>     80:30853/TCP   5m12s
service/app-crud-mongodb        ClusterIP      10.111.146.0     <none>        27017/TCP      5m12s
service/kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP        36d

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-crud-express-crud   1/1     1            1           5m12s
deployment.apps/app-crud-mongodb        1/1     1            1           5m12s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/app-crud-express-crud-565b97d46d   1         1         1       5m12s
replicaset.apps/app-crud-mongodb-5cc7c8c7d7        1         1         1       5m12s
~~~

A través del puerto externo del servicio de Loadbalancer se accede a la aplicación.

![appCRUD](image/appCRUD.png)

![addCRUD](image/addCRUD.png)

![listCRUD](image/listCRUD.png)

-----------------------

Dos grandes beneficios que se obtienen al utilizar Helm es poder modificar el chart y actualizarlo de forma muy sencilla con el comando `helm upgrade` y volver a un estado de la aplicación anterior, es decir, realizar un rollback a la versión del chart que queramos con el comando `helm rollback`. A continuación, se explicará esta última utilidad.

### Actualizar el chart

Se va a realizar una modificación en la aplicación, aumentar las réplicas. Para ello se edita el valor de `replicaCount` en el fichero `values.yaml`.

###### Antes
````yaml
replicaCount: 1
````
###### Después
````yaml
replicaCount: 2
````

Para que se realicen los cambios, creándose una versión nueva del chart, se ejecuta el comando `helm update`.

Antes deactualizar la versión del chart es 1:

***debian@cliente:**~/app-crud* **$** `helm list`
~~~~
NAME      NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
app-crud  default         1               2020-12-04 13:54:08.201085786 +0000 UTC deployed        app-crud-0.1.0  1.0.1
~~~~

Y al actualizar se muestra un mensaje indicando que la versión del chart se ha actualizado:

***debian@cliente:**~/app-crud* **$** `helm upgrade app-crud ./`
~~~~
Release "app-crud" has been upgraded. Happy Helming!

NAME: app-crud
LAST DEPLOYED: Fri Dec  4 13:57:45 2020
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get svc -w app-crud-app-crud'
  export SERVICE_IP=$(kubectl get svc --namespace default app-crud-app-crud -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP:80
~~~~

***debian@cliente:**~/app-crud* **$** `helm list`
~~~~
NAME      NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
app-crud  default         2               2020-12-04 13:57:45.495586135 +0000 UTC deployed        app-crud-0.1.0  1.0.1
~~~~

También se puede comprobar el cambio con el comando `helm get values`.

***debian@cliente:**~/app-crud* **$** `helm get values --all app-crud | egrep replica`
````yaml
replicaCount: 2
````

Además, se ha creado el nuevo pod. Se comprueba listando los objetos pod que hay en el cluster:

***debian@cliente:**~/app-crud* **$** `kubectl get pod`
~~~~
NAME                                     READY   STATUS            RESTARTS   AGE
app-crud-express-crud-565b97d46d-knwkh   1/1     Running           0          3m50s
app-crud-express-crud-757b4f78fb-tn5d4   0/1     PodInitializing   0          13s
app-crud-mongodb-5bc9679bcc-vk6qh        1/1     Running           0          3m50s
~~~~


### Realizar un Rollback a una version anterior del chart

A continuación, se va a explicar como volver a una versión anterior del chart. En esta práctica, de la versión 2, donde la aplicación contiene dos pods replicados, a la versión 1. Para ello se hace uso del comando `helm upgrade` seguido de dos argumentos. El primero de ellos será el nombre de una versión y el segundo es un número de revisión

Para ver los números de revisión, se ejecuta el comando `helm history`

***debian@cliente:**~/app-crud* **$** `helm history app-crud`
~~~~
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Fri Dec  4 13:54:08 2020        superseded      app-crud-0.1.0  1.0.1           Install complete
2               Fri Dec  4 13:57:45 2020        deployed        app-crud-0.1.0  1.0.1           Upgrade complete
~~~~

Se vuelve a la revisión 1:

***debian@cliente:**~/app-crud* **$** `helm rollback prueba 1`
~~~~
Rollback was a success! Happy Helming!
~~~~

Y al listar las revisiones de nuevo:

***debian@cliente:**~/app-crud* **$** `helm history app-crud`
~~~~
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Fri Dec  4 13:54:08 2020        superseded      app-crud-0.1.0  1.0.1           Install complete
2               Fri Dec  4 13:57:45 2020        superseded      app-crud-0.1.0  1.0.1           Upgrade complete
3               Fri Dec  4 14:06:43 2020        deployed        app-crud-0.1.0  1.0.1           Rollback to 1
~~~~

Mostrar el valor del atributo réplica:

***debian@cliente:**~/app-crud* **$** `helm get values --all app-crud | egrep replica`
````yaml
replicaCount: 1
````

Además, se observa que el nuevo pod está en estado `Terminating`.

***debian@cliente:**~/app-crud* **$** `kubectl get pod`
~~~~
NAME                                     READY   STATUS            RESTARTS   AGE
app-crud-express-crud-565b97d46d-knwkh   1/1     Running           0          12m
app-crud-express-crud-757b4f78fb-tn5d4   0/1     Terminating       0          16s
app-crud-mongodb-5bc9679bcc-vk6qh        1/1     Running           0          12m
~~~~

-----------------------

### Despliegue de Chart de aplicación PHP utilizando Laravel y MySQL

Se va desplegar una nueva aplicación PHP de Larabel 7, frameworks de aplicaciones PHP de código abierto. Implementado con una base de datos MySQL con la ayuda del chart oficial `stable/lamp`.

Para el despliegue de esta aplicación se va a utilizar una imagen docker propia, `moralg/larabel-kubernetes`, creada con docker compose y subida al repositorio [DockerHub/moralg](https://hub.docker.com/repository/docker/moralg/laravel-kubernetes).

> #### NOTA
> ----------------------
> La aplicación de larabel se ha construido con ayuda del repositorio oficial de Larabel en [Github](https://github.com/laravel/laravel).

Antes de realizar la construcción de la imagen, se ha añadido una modificación al fichero `larabel/resources/views/welcome.blade.php`, para probar la conexión de la base de datos y mostrar el nombre si realiza la conexión.

````html
<div class="links">
   <strong>Database Connected: </strong>
    @php
        try {
            DB::connection()->getPDO();
            echo DB::connection()->getDatabaseName();
            } catch (\Exception $e) {
            echo 'None';
        }
    @endphp
</div>
````

Para conocer las variables que más tarde se utilizarán para la creación de los ficheros `values.yaml`, datos no confidenciales, y `secrets.yaml`, datos confidenciales, es importante el código del docker compose con el que se ha realizado la imagen, que es el siguiente:

````yaml
version: '3.5'
services:
  php:
    image: moralg/laravel-kubernetes:latest
    restart: always
    ports:
      - 8000:80
    environment:
      - APP_KEY=base64:8thPzqcZG1NC8ga7TYrXQqEBXozhK5RgMqPVltVr9Ig=
      - APP_ENV=local
      - APP_DEBUG=true
      - DB_PORT=3306
      - DB_HOST=mysql
      - DB_DATABASE
      - DB_USERNAME
      - DB_PASSWORD
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_DATABASE}
      - MYSQL_USER=${DB_USERNAME}
      - MYSQL_PASSWORD=${DB_PASSWORD}
````

> #### NOTA
> ----------------------
> Además hay que tener en cuenta que el chart ofcial que se va a utilizar tiene una serie de valores que se pueden modificar. 
> 
> En el repositorio de GitHub [stable/lamp](https://github.com/helm/charts/tree/master/stable/lamp) se pueden mirar los valores configurables.

Se crea el fichero `values.yaml` con el siguiente código donde se han añadido los valores `repository` y `tag`, para indicar la imagen de docker de la aplicación Larabel. También se han indicado las variables de entorno no confidenciales.

````yaml
php:
  repository: "moralg/laravel-kubernetes"
  tag: "latest"
  fpmEnabled: false
  envVars:
    - name: APP_ENV
      value: pro
    - name: APP_DEBUG
      value: false
    - name: DB_PORT
      value: 3306
    - name: DB_HOST
      value: localhost
````

Se crea el fichero `secrets.yaml` con el siguiente código donde se han añadido las variables confidenciales de la base de datos MySQL y, con los mismos valores, a la aplicación php, para que tenga acceso a MySQL.

````yaml
mysql:
  rootPassword: "bd_root_pass"
  user: bd_user
  password: "bd_pass"
  database: bd_name

php:
  envVars:
    - name: APP_KEY
      value: "base64:8thPzqcZG1NC8ga7TYrXQqEBXozhK5RgMqPVltVr9Ig="
    - name: DB_DATABASE
      value: bd_name
    - name: DB_USERNAME
      value: bd_user
    - name: DB_PASSWORD
      value: "bd_pass"
````

Creado los dos ficheros se instala el chart con el comando `helm install` y añadiendo el parámetro `-f` para indicar los ficheros, además de el chart oficial `stable/lamp`

***debian@cliente:**~/app-larabel* **$** `helm install laravel-kubernetes -f values.yaml -f secrets.yaml stable/lamp`
~~~~
NAME: laravel
LAST DEPLOYED: Fri Dec  4 12:11:24 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
INIT:
      Please wait for all init containers to finish before connecting to
      the charts services. This might take a few minutes depending on their
      tasks.

LOADBALANCER:
      Please wait until the service has been routed to an IP address.
      You can watch the status of by running 'kubectl get svc -w laravel-lamp'

1. You can now connect to the following services:
      export CHARTIP=$(kubectl get svc laravel-lamp --output=jsonpath={.status.loadBalancer.ingress..ip})

      Main Site:
        http://$CHARTIP
~~~~

> #### NOTA
> ----------------------
> Para la base de datos MySQL, es necesario crea un Persistent Volume para guardar los datos. A continuación, un ejemplo del fichero de creación de un PV:  
> ````yaml
> apiVersion: v1
> kind: PersistentVolume
> metadata:
>   name: volume-mysql
>   labels:
>     type: local
> spec:
>   capacity:
>     storage: 10Gi
>   accessModes:
>     - ReadWriteOnce
>   hostPath:
>     path: "/mnt"
> ````
> Y se ejecuta lo siguiente:
> 
> ***debian@cliente:**~* **$** `kubectl app install laravel-kubernetes -f values.yaml -f secrets.yaml stable/lamp`

Las comprobaciones se realizan como en la práctica anterior, listando los objetos del cluster de Kubernetes:

***debian@cliente:**~* **$** `kubectl get all`
~~~~
NAME                                READY   STATUS    RESTARTS   AGE
pod/laravel-lamp-68dffb6495-9hlx6   2/2     Running   0          48m

NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
service/kubernetes     ClusterIP      10.96.0.1       <none>        443/TCP                       40d
service/laravel-lamp   LoadBalancer   10.103.15.225   <pending>     80:32418/TCP,3306:30419/TCP   48m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/laravel-lamp   1/1     1            1           48m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/laravel-lamp-68dffb6495   1         1         1       48m
~~~~

Para acceder a la aplicación desde http, se utiliza la ip del servidor y el puerto externo asignado al httpd, en este caso el **32418**.

![app-larabel](image/app-larabel.png)

----------

### Creación de un repositorio público

Se pueden compartir charts de Helm a través de un repositorio público. Alojando los chart, gestionados por un fichero `index.yaml` para las diferentes releases, compuesto de metadatos y enlaces a los chart de Helm.

De esta forma se puede utlizar Github Pages, S3 de AWS, Almacenamiento en la nube de Google, etc. En esta práctica se va a utilizar Github Pages.

Se crea un repositorio en Github, [Public_repository_helm](https://github.com/MoralG/Public_repository_helm/tree/repo-helm).

![Create_repository](image/Create_repository.png)

Se clona el repositorio en nuestro equipo:

***debian@cliente:**~* **$** `git clone git@github.com:MoralG/Public_repository_helm.git`
~~~~
Cloning into 'Public_repository_helm'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
~~~~

El siguiente paso es añadir algunos charts de Helm. Se va a utilizar los chart que se crean por defecto al utilizar `helm create`.

***debian@cliente:**~/Public_repository_helm* **$** `mkdir charts`

***debian@cliente:**~/Public_repository_helm/charts* **$** `helm create chart1 && helm create chart2`
~~~~
Creating chart1
Creating chart2
~~~~

***debian@cliente:**~/Public_repository_helm/charts* **$** `tree`
~~~~
.
├── chart1
│   ├── charts
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   ├── _helpers.tpl
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── NOTES.txt
│   │   ├── serviceaccount.yaml
│   │   ├── service.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
└── chart2
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   ├── serviceaccount.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

8 directories, 20 files
~~~~

Se va crear una rama vacía, `repo-helm`:

***debian@cliente:**~/Public_repository_helm* **$** `git checkout --orphan repo-helm`
~~~~
Switched to a new branch 'repo-helm'
~~~~

Se añade a la rama `repo-helm` los charts: 

***debian@cliente:**~/Public_repository_helm* **$** `git add *`

***debian@cliente:**~/Public_repository_helm* **$** `git commit -m "Charts añadidios"`
~~~~
[repo-helm (root-commit) 5a8be27] Charts añadidios
 Committer: Debian <debian@cliente.novalocal>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 23 files changed, 762 insertions(+)
 create mode 100644 README.md
 create mode 100644 charts/chart1/.helmignore
 create mode 100644 charts/chart1/Chart.yaml
 create mode 100644 charts/chart1/templates/NOTES.txt
 create mode 100644 charts/chart1/templates/_helpers.tpl
 create mode 100644 charts/chart1/templates/deployment.yaml
 create mode 100644 charts/chart1/templates/hpa.yaml
 create mode 100644 charts/chart1/templates/ingress.yaml
 create mode 100644 charts/chart1/templates/service.yaml
 create mode 100644 charts/chart1/templates/serviceaccount.yaml
 create mode 100644 charts/chart1/templates/tests/test-connection.yaml
 create mode 100644 charts/chart1/values.yaml
 create mode 100644 charts/chart2/.helmignore
 create mode 100644 charts/chart2/Chart.yaml
 create mode 100644 charts/chart2/templates/NOTES.txt
 create mode 100644 charts/chart2/templates/_helpers.tpl
 create mode 100644 charts/chart2/templates/deployment.yaml
 create mode 100644 charts/chart2/templates/hpa.yaml
 create mode 100644 charts/chart2/templates/ingress.yaml
 create mode 100644 charts/chart2/templates/service.yaml
 create mode 100644 charts/chart2/templates/serviceaccount.yaml
 create mode 100644 charts/chart2/templates/tests/test-connection.yaml
 create mode 100644 charts/chart2/values.yaml
~~~~

***debian@cliente:**~/Public_repository_helm* **$** `git push origin repo-helm`
~~~~
Enumerating objects: 31, done.
Counting objects: 100% (31/31), done.
Delta compression using up to 2 threads
Compressing objects: 100% (29/29), done.
Writing objects: 100% (31/31), 6.28 KiB | 3.14 MiB/s, done.
Total 31 (delta 10), reused 1 (delta 0)
remote: Resolving deltas: 100% (10/10), done.
remote:
remote: Create a pull request for 'repo-helm' on GitHub by visiting:
remote:      https://github.com/MoralG/Public_repository_helm/pull/new/repo-helm
remote:
To github.com:MoralG/Public_repository_helm.git
 * [new branch]      repo-helm -> repo-helm
~~~~

Y se habilita Github Pages en los ajustes del repositorio. La dirección es `Settings > Options > GitHub Pages`.

Se indica la rama `repo-helm`, el directorio raíz, `/(root)`.

![github-pages](image/github-pages.png)

Tras unos segundos, ya se podrá acceder a la URL que nos proporciona Github Pages.

![github-pages-web](image/github-pages-web.png)

Se puede utilizar un combinación de los comandos `helm package` y `helm repo` para construir el repositorio de Helm a mano, o se puede utilizar `chart-releaser` que, además de crear los paquetes, éstos se cargaran como binarios en una release de Github, con la versión adecuada.

Se descarga la última versión de `chart-releaser` desde el repositorio de [Helm en Github](https://github.com/helm/chart-releaser/releases) y se descomprime el fichero con el comando `tar`.

***debian@cliente:**/tmp* **$** `curl -sSL https://github.com/helm/chart-releaser/releases/download/v0.2.1/chart-releaser_0.2.1_linux_amd64.tar.gz | tar xzf -`

Se mueve el del binario al directorio  `/bin/cr`

***debian@cliente:**/tmp* **$** `sudo mv cr /bin/cr`

Se comprueba que está instalado y la versión:

***debian@cliente:**~* **$** `cr version`
~~~~
Version:         v0.2.1
Git commit:      5132b262ef33b074bf47ce8ee446433849e93873
Date:            2019-05-23T06:19:20Z
License:         Apache 2.0
~~~~

Dos comando importantes: `cr index` (creará un fichero `index.yaml`) y `cr upload` (cargará los paquetes a las release de Github). Para esto último, se necesita un token de Github para que pueda utilizar la API.

La dirección para crear un token es: `Settins > Developer settings > Personal access tokens > Generate new token`, se asigna un nombre y se marcan los ámbitos: `repo`, `workflow`, `write:packages`, `delete:packages`.

Se crea una variable de entorno con el token de acceso.

***debian@cliente:**~* **$** ` export CR_TOKEN=b67e55c97bcaa440d75de4258a2e79b551101c5f`

El siguiente paso será crear y cargar los paquetes, para esto se utiliza el comndo `helm package`. Hay que crear un directorio para los paquetes que se van a cargar, además de añadir este directorio al fichero `.gitignore` para que no se suba al repositorio.

***debian@cliente:**~/Public_repository_helm* **$** `echo ".deploy" >> .gitignore`

***debian@cliente:**~/Public_repository_helm* **$** `helm package charts/{chart1,chart2} --destination .deploy`
~~~~
Successfully packaged chart and saved it to: .deploy/chart1-0.1.0.tgz
Successfully packaged chart and saved it to: .deploy/chart2-0.1.0.tgz
~~~~

Se obseva que se han creado dos paquetes. El nombre de los paquetes se ha generado con la versión que tenemos asignada en el fichero `Chart.yaml`, esta versión también será la misma que la del `tag` de la release de Github.

***debian@cliente:**~/Public_repository_helm* **$** `ls .deploy/`
~~~~
chart1-0.1.0.tgz  chart2-0.1.0.tgz
~~~~

Se ejecuta `cr upload` para crear versiones y cargar los paquetes.

***debian@cliente:**~/Public_repository_helm* **$** `cr upload -o moralg -r Public_repository_helm -p .deploy -t <<<TOKEN>>>`

Con este comando se indica el propietario del repositorio, `-o`, el nombre del repositorio, `-r`, el directorio donde se almacenan los paquetes, `-p`, y el token de acceso `-t`.

Al ejecutar el comando, no devuelve ninguna salida. Si todo ha salido bien, el repositorio tendrá dos release.

![release](image/release.png)

Se continua con la creación del fichero `index.yaml` para posteriormente subirlo. 

Se selecciona la rama `repo-helm`:

***debian@cliente:**~/Public_repository_helm* **$** `git checkout repo-helm`
~~~~
Already on 'repo-helm'
~~~~

Se crea el fichero `index.yaml` añadiendo los datos del propietario y del repositorio.

***debian@cliente:**~/Public_repository_helm* **$** `cr index -i ./index.yaml -p .deploy -o moralg -r Public_repository_helm`
~~~~
====> UpdateIndexFile new index at ./index.yaml
====> Found chart1-0.1.0.tgz
====> Extracting chart metadata from .deploy/chart1-0.1.0.tgz
====> Calculating Hash for .deploy/chart1-0.1.0.tgz
====> Found chart2-0.1.0.tgz
====> Extracting chart metadata from .deploy/chart2-0.1.0.tgz
====> Calculating Hash for .deploy/chart2-0.1.0.tgz
--> Updating index ./index.yaml
~~~~

El comando anterior crea el fichero `index.yaml` y el contenido automáticamente con los detalles de los charts y la ruta de sus ficheros.

~~~~
apiVersion: v1
entries:
  chart1:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2020-12-07T14:20:15.230371116Z"
    description: A Helm chart for Kubernetes
    digest: d3f63b3a4fd81e77ed84833a0306a951c4d3003a5c718d3a1ab5d9ecdd59c8c5
    name: chart1
    urls:
    - https://github.com/MoralG/Public_repository_helm/releases/download/chart1-0.1.0/chart1-0.1.0.tgz
    version: 0.1.0
  chart2:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2020-12-07T14:20:15.277111561Z"
    description: A Helm chart for Kubernetes
    digest: 04647e714ac8ab7e63efa32a7cd5bec7186af3b086bc50aa98cf4dfaee874aa2
    name: chart2
    urls:
    - https://github.com/MoralG/Public_repository_helm/releases/download/chart2-0.1.0/chart2-0.1.0.tgz
    version: 0.1.0
generated: "2020-12-07T14:20:14.977062307Z"
~~~~

Se sube el fichero `index.yaml` al repositorio y se comprueba que existe en Github Pages.

![release_web](image/release_web.png)

Comprobación del nuevo repositorio: 

Se añade el repositorio con el comando `helm repo add` y se utiliza el nombre de nuestro repositorio y la dirección de Github Pages.

***debian@cliente:**~* **$** `helm repo add public_repository_helm https://moralg.github.io/Public_repository_helm/`
~~~~
"public_repository_helm" has been added to your repositories
~~~~

Se lista los repositorios:

***debian@cliente:**~* **$** `helm repo list`
~~~~
NAME                    URL
django                  https://itswcg.github.io/django-helm/charts/
bitnami                 https://charts.bitnami.com/bitnami
stable                  https://charts.helm.sh/stable
public_repository_helm  https://moralg.github.io/Public_repository_helm/
~~~~

El repositorio se ha creado y se ha añadido a Helm. Lo último es instalar uno de los chart que se ha añadido al repositorio.

***debian@cliente:**~* **$** `helm install test public_repository_helm/chart1`

~~~~
NAME: test
LAST DEPLOYED: Mon Dec  7 14:34:08 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=chart1,app.kubernetes.io/instance=test" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:80
~~~~

***debian@cliente:**~* **$** `kubectl get pod`
~~~~
NAME                               READY   STATUS    RESTARTS   AGE
test-chart1-7979df9f94-nv5kk       1/1     Running   0          97s
~~~~

## Siguientes Pasos

Los siguientes pasos que se podrían realizar para ampliar esta práctica sería la utilización de Helmfile. Esta herramienta permite declarar una definición de un cluster de Kubernetes completo en un solo fichero YAML, agrupar múltiples versiones de Helm, instalación de chart de Helm, y permite ajustar una especificación de cada versión en función del tipo de entorno que se necesite (desarrolo, prueba, producción).

También se podría automatizar las actualizaciones de los chart de Helm para nuestro repositorio con [CicleCI](https://circleci.com/). Es un servicio de integración continua muy utilizado por la comunidad de Helm.

## Conclusiones

Una vez controlado las partes del código, escritos en Go, de los ficheros YAML, donde se indican las variables que utlizará el fichero `values.yaml`, el proceso de gestionar una aplicación en charts a través de Helm es más cómodo para realizar un cambio y actualizar la versión. También es más sencillo revertir cualquier fallo o configuración no deseada con rollback.

Además, Helm y su comunidad está muy implicada en el desarrolo de creación de chart para la utilización pública.


## Webgrafía

Helm. The Chart Template Developer's Guida.

https://helm.sh/docs/chart_template_guide/


Helm. Quickstart Guide.

https://helm.sh/docs/intro/quickstart/

Helm. Installing Helm.

https://helm.sh/docs/intro/install/

Helm. Using Helm.

https://helm.sh/docs/intro/using_helm/

Helm. The Chart Best Practices Guide
.
https://helm.sh/docs/chart_best_practices/

Helm. Helm Command.

https://helm.sh/docs/helm/

Jfrog. Steering Straight with Helm Chart Best Practices.

https://jfrog.com/blog/helm-charts-best-practices/

DigitalOcean. Cómo implementar Laravel7 y MySQL en Kubernetes con Helm.

https://www.digitalocean.com/community/tutorials/how-to-deploy-laravel-7-and-mysql-on-kubernetes-using-helm-es

Tech. Creating a Helm Chart Repository - Part 1

https://tech.paulcz.net/blog/creating-a-helm-chart-monorepo-part-1/