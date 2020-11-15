# Despliegues de microservicios en k8s con Helm (y OpenShift)

Helm es un administrador de paquetes para Kubernetes, que ayuda en el proceso de gestión de versiones a desplegar, su empaquetado, proceso de release (forward, rollback, upgrade), etc... de una manera más fácil y rápida.

Estos paquetes se denominan chart, los cuales son una colección de ficheros que describen un conjunto de recursos del API de Kubernetes. Un ejemplo comparable para entenderlo sería el caso del `apt` o `yum` o otros administradores de paquetes de distribuciones de linux pero para Kubernetes.

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

* Un clúster de Kubernetes en la versión 1.8 o posterior, con el control de acceso en roles (RBAC) hablitado.

###### [Instalación Kubernetes con Kubeadm](https://github.com/MoralG/Trabajando_con_Kubernetes/blob/master/Trabajando_con_Kubernetes.md)

###### Comprobar la versión de Kubernetes:
~~~
kubectl version
    Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
    Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2",   GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean",   BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
~~~

--------------------------------------

* Tener la herramienta de líneas de comando `kubectl` instalada en su equipo local, configurada para poder conectarse al clúster.

###### Comprobar la conectividad:
~~~
kubectl cluster-info
    Kubernetes master is running at https://10.0.0.10:6443
    KubeDNS is running at https://10.0.0.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
~~~

----------------

## Instalación de Helm

Vamos a instala la herramienta de linea de comando `helm` en nuestro equipo local de trabajo.

Descargamos la última release de Helm de la versión 3. Para hacer esto nos vamos a la [página oficial](https://github.com/helm/helm/releases) y nos descargamos el fichero `.tar.gz` para Linux

> **NOTA**: Es recomendable que nos descarguemos una versión que este **Verificada**.

En el momento de la creación de este tutorial, nos descargamos la versión 3.2.0 de Helm.
~~~
wget https://get.helm.sh/helm-v3.3.4-linux-amd64.tar.gz
~~~

Descromprimimos el fichero:
~~~
tar -zxvf helm-v3.3.4-linux-amd64.tar.gz
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

## Guía rápida

### Instalando un Chart oficial

Vamos a instalar un chart del repositorio oficial de Helm, para hacer esto tenemos que actualizar primero la información de los chart disponibles localmente, para descargar, si ha habido cambios.

~~~
helm repo update
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "stable" chart repository
    Update Complete. ⎈ Happy Helming!⎈
~~~

No ha salido un mensaje de que esta todo actualizado, ahora vamos a proceder a instalar un chart. En nuestro caso vamos a probar con MySQL.

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

Vamos a asignarle el nombre de **maria** a nuestro chart, pero podemos utilizar la flags `--generate-name`para asignarle uno automáticamente.

~~~
helm install maria stable/mysql
~~~

Al instalar dicho chart, en el caso de mysql, nos sale una información del chart, como se muestra a continuación:

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

> **NOTA**: Si queremos ver la opciones configurables de un chart, podemos usar `helm show values <nombre_chart>`
> ~~~
> helm show values stable/mariadb
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
> Sabiendo las opciones que podemos configurar, podemos modificarlar de dos maneras:
> * Indicandole los parámetros en un fichero `.yaml` y luego indicarle dicho fichero en la instalación del chart.
>
> Creamos el fichero `yaml` indicandole las opciones
> ~~~
> echo '{mariadbUser: usuario1, mariadbDatabase: usuario_bd}' > prueba.yaml
> ~~~
> Indicamos el fichero `yaml` en la instalación, con el parámetro `-f`
> ~~~
> helm install -f prueba.yaml stable/mariadb --generate-name
> ~~~
> * Le indicamos las opciones con el parametro `--set`, en el moemnto de la instalación
> ~~~
> helm install stable/mariadb --generate-name --set name=usuario1
> ~~~

Ya tenemos instalado nuestro chart y nos muestra varia información, como la descripción, el nombre, el namespace, etc. Pero además nos muestra la instrucciones para conectarse a la base de datos.

Para ver los chart que estan lanzados con Helm podemos utilizar el comando `helm ls`.


~~~
helm ls
    NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP     VERSION
    maria	default  	1       	2020-04-23 17:39:05.877933281 +0000 UTC	deployed	mysql-1.6.3	5.7.28
~~~
### Actualizando un versión

Cuando se lanza una nueva versión de un chart, cuando desea cambiar la configuración de este, podemos usar `helm upgrade`

Una actualización toma una versión existente y la actualiza de acuerdo con la información que proporciones, en el caso de Helm solo se actualiza las cosas qie no han cambiado desde la última versión.

En este caso, para ver un ejemplo, vamos a cambiar la configuración del chart instalado anteriormente. Vamos a crear un fichero `yaml` con la opción `mariadbUser` modificada y creareamos una nueva versión.

~~~
echo '{mariadbUser: usuario1}' > prueba.yaml
~~~

~~~
helm upgrade -f prueba.yaml maria stable/mariadb
~~~

~~~
helm get values maria
    USER-SUPPLIED VALUES:
    mariadbUser: usuario1
~~~

~~~
helm history maria
    REVISION	UPDATED                 	STATUS     	CHART         	APP VERSION	DESCRIPTION
    1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
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
    1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
    2       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	    5.7.28     	Uninstallation complete
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
    1       	Thu Apr 23 16:30:03 2020	deployed   	mysql-1.6.3   	5.7.28    	Upgrade complete
    2       	Thu Apr 23 17:39:05 2020	uninstalled	mysql-1.6.3	    5.7.28     	Uninstallation complete
    3       	Thu Apr 23 17:51:41 2020	deployed	mysql-1.6.3	    5.7.28     	Uninstallation complete
~~~

## Creando Charts en Helm

En esta guía vamos a desarrollar nuestros propios Charts en Helm. Es recomentable que se mire antes la [Instalación y configuración](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm_y_OpenShift/blob/master/Proyecto.md#instalaci%C3%B3n-de-helm) de Helm y la [Guía rápida](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm_y_OpenShift/blob/master/Proyecto.md#gu%C3%ADa-r%C3%A1pida).

Si quiere saber el funcionamiento de algunos comando de Helm puede ir a la [Guía de comando]() de Helm.

Un Chart es una colección de ficheros que describen un cojunto de recursos de Kubernetes. Podemos usar un solo chart para implementar un pod memcached, o si nos vamos a algo mas complicado, una pila completa de aplicaciones web con servidores HTTP, bases de datos, cache, etc.

Es bueno tener unas prácticas recomendadas para realizar chart de Helm, por eso vamos a ir paso a paso creando una aplicación en python y dando recomendaciones.

Vamos a empezar creando el chart:

> **debian@cliente:***~* **$** ``helm create appPython3``
~~~
Creating appPython3
~~~

Al crear un Chart, dispondremos de unos directorios en forma de árbol, los cuales podemos modificar y una vez terminado la modificación, empaquetarlo en archivos versionados para su implementación.

~~~
appPython3/
├── charts
|   └──
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

3 directories, 10 files
~~~

Vamos a repasar los ficheros y directorios necesarios para realizar un buen despligue en Kubernetes.

|Objeto          | Descripción
|----------------|-------------------------------
|**Chart.yaml**| Fichero yaml que contiene la información del chart.
|**README.md**| Fichero utilizado para la descripción del chart.
|**values.yaml**| Fichero de configuración de los distintos valores del chart.
|**templates/**| Directorio donde se almacenan las plantillas y donde se generará ficheros de Kubernetes.
|**templates/NOTES.txt**| Fichero opcional sin formato, que contiene breves notas de uso.
|**dependences.yaml**| Dichero que no se genera al crear el chart pero que es recomendable tener si vamos a ñadir dependecias.

Vamos a crear un repositorio en Github

**debian@cliente:***~/appPython3* **$** `helm repo index .`

**debian@cliente:***~/appPython3* **$** `ls`
~~~
charts  Chart.yaml  index.yaml  templates  values.yaml
~~~

**debian@cliente:***~/appPython3* **$** `cat index.yaml `
~~~
apiVersion: v1
entries: {}
generated: "2020-11-12T10:09:54.715010553Z"
~~~

**debian@cliente:***~/appPython3* **$** `git init`
~~~
Initialized empty Git repository in /home/debian/appPython3/.git/
~~~

**debian@cliente:***~/appPython3* **$** `echo "Aplicación en python con base de dato Postgres creada con Helm para Kubernetes." > README.md`

**debian@cliente:***~/appPython3* **$** `git branch -M master`

**debian@cliente:***~/appPython3* **$** `git remote add origin https://github.com/MoralG/appPython3.git`

**debian@cliente:***~/appPython3* **$** `git add *`

**debian@cliente:***~/appPython3* **$** `git commit -m "Generar repositorio para Helm"`
~~~
[master 3cfa144] Generar repositorio para Helm
 Committer: Debian <debian@cliente.novalocal>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 12 files changed, 360 insertions(+)
 create mode 100644 Chart.yaml
 create mode 100644 index.yaml
 create mode 100644 templates/NOTES.txt
 create mode 100644 templates/_helpers.tpl
 create mode 100644 templates/deployment.yaml
 create mode 100644 templates/hpa.yaml
 create mode 100644 templates/ingress.yaml
 create mode 100644 templates/service.yaml
 create mode 100644 templates/serviceaccount.yaml
 create mode 100644 templates/tests/test-connection.yaml
 create mode 100644 values.yaml
 create mode 100644 README.md
~~~

**debian@cliente:***~/appPython3* **$** `git push -u origin master`
~~~
Username for 'https://github.com': moralg
Password for 'https://moralg@github.com': 
Enumerating objects: 16, done.
Counting objects: 100% (16/16), done.
Delta compression using up to 2 threads
Compressing objects: 100% (14/14), done.
Writing objects: 100% (15/15), 5.14 KiB | 2.57 MiB/s, done.
Total 15 (delta 0), reused 0 (delta 0)
To https://github.com/MoralG/appPython3.git
   ed95ff7..3cfa144  master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
~~~

**debian@cliente:***~/appPython3* **$** `helm repo add my-repo https://raw.githubusercontent.com/moralg/appPython3/master`
~~~
"my-repo" has been added to your repositories
~~~

**debian@cliente:***~/appPython3* **$** `helm repo update`
~~~
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "my-repo" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
~~~

**debian@cliente:***~/appPython3* **$** `helm repo list`
~~~
NAME   	URL                                                       
stable 	https://kubernetes-charts.storage.googleapis.com/         
my-repo	https://raw.githubusercontent.com/moralg/appPython3/master
~~~

A partir de aquí tenemos que preguntarnos que vamos a necesitar.

* ¿Que imagen o imagenes vamos a utilizar?
* ¿Que dependencias necesitamos definir?
* ¿Para nuestra aplicación necesitamos volumenes persistentes?

Para empezar vamos a modificar los metadatos del fichero `Chart.yaml`.

```yaml
apiVersion: v1
appVersion: 1.0.0
name: appPython3
description: Aplicación en python con base de datos Postgresql
version: 0.1.0
type: application
sources: 
- https://github.com/MoralG/appPython3
maintainers:
- name: moralg
  email: ale95mogra@gmail.com
icon: https://www.quytech.com/blog/wp-content/uploads/2020/06/python-web-development.jpg
```

Ahora vamos a definir las dependencias, dado que nuestra aplicación necesita la base de datos postgresql, debemos especificarla en la lista de dependencias del fichero `requirement.yaml`. Este lo tenemos que crear en el directorio de nuestro chart.

**debian@cliente:***~/appPython3* **$** `touch dependeces.yaml`
```yaml
dependencies:
- name: postgresql-ha #Nombre del chart de postgresql, debe coincidir con el nombre del fichero Chart.yaml de ese chart.
  version: 11.9.0 #Versión del chart que vamos a utilizar.
  repository: https://charts.bitnami.com/bitnami #Repositiorio del que vamos a obtener postgresql.
  condition: postgresql-ha.enabled #Condición para que este activa la dependencia antes que la app.
```

**debian@cliente:***~/appPython3* **$** `helm dep update`

Lo siguiente que tenemos que hacer es modificar un poco el fichero `deployment.yaml` que se ha creado con el comando `helm create`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "appPython3.fullname" . }}
  labels:
    app: {{ template "appPython3.name" . }}
    chart: {{ template "appPython3.chart" . }}
    release: {{ .Release.Name }}
spec:
{{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
{{- end }}
  selector:
    matchLabels:
      {{- include "appPython3.selectorLabels" . | nindent 6 }}
  template:
    metadata:
    {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      labels:
        {{- include "appPython3.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "appPython3.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: {{ .Values.service.name }}
              containerPort: {{ .Values.service.internalPort }}
              protocol: {{ .Values.service.protocol }}
          readinessProbe:
            httpGet:
              path: {{ .Values.readinessProbe.path }}
              port: {{ default .Values.service.name .Values.readinessProbe.port }}
            initialDelaySeconds: {{ .Values.readinessProbe.initialDelaySeconds }}
            periodSeconds: {{ .Values.readinessProbe.periodSeconds }}
            timeoutSeconds: {{ .Values.readinessProbe.timeoutSeconds }}
            successThreshold: {{ .Values.readinessProbe.successThreshold }}
            failureThreshold: {{ .Values.readinessProbe.failureThreshold }}
          livenessProbe:
            httpGet:
              path: {{ .Values.livenessProbe.path }}
              port: {{ default .Values.service.name .Values.livenessProbe.port }}
            initialDelaySeconds: {{ .Values.livenessProbe.initialDelaySeconds }}
            periodSeconds: {{ .Values.livenessProbe.periodSeconds }}
            timeoutSeconds: {{ .Values.livenessProbe.timeoutSeconds }}
            successThreshold: {{ .Values.livenessProbe.successThreshold }}
            failureThreshold: {{ .Values.livenessProbe.failureThreshold }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```
Añadimos algunas label, para tener un control y una comodidad a la hora de listar y trabajar con los objetos que vamos a crear
```yaml
labels:
  app: {{ template "appPython3.name" . }}
  chart: {{ template "appPython3.chart" . }}
  release: {{ .Release.Name }}
```

Cambiamos la el parámetro image para actualizar el chart de Helm con una nueva versión de la aplicación simplemente cambiando el valor en Chart.yaml.
```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```
Añadimos las siguientes lineas para podemos modificar la información de los puertos internos desde el fichero `values.yaml`
```yaml
ports:
  - name: {{ .Values.service.name }}
    containerPort: {{ .Values.service.internalPort }}
    protocol: {{ .Values.service.internalProtocol }}
```

Siempre es bueno agregar una prueba de preparación y de vivacidad para verificar el estado continuo de la aplicación. Para esto se utiliza las sondas.
```yaml
readinessProbe:
  httpGet:
    path: {{ .Values.readinessProbe.path }}
    port: {{ default .Values.service.name .Values.readinessProbe.port }}
  initialDelaySeconds: {{ .Values.readinessProbe.initialDelaySeconds }}
  periodSeconds: {{ .Values.readinessProbe.periodSeconds }}
  timeoutSeconds: {{ .Values.readinessProbe.timeoutSeconds }}
  successThreshold: {{ .Values.readinessProbe.successThreshold }}
  failureThreshold: {{ .Values.readinessProbe.failureThreshold }}
livenessProbe:
  httpGet:
    path: {{ .Values.livenessProbe.path }}
    port: {{ default .Values.service.name .Values.livenessProbe.port }}
  initialDelaySeconds: {{ .Values.livenessProbe.initialDelaySeconds }}
  periodSeconds: {{ .Values.livenessProbe.periodSeconds }}
  timeoutSeconds: {{ .Values.livenessProbe.timeoutSeconds }}
  successThreshold: {{ .Values.livenessProbe.successThreshold }}
  failureThreshold: {{ .Values.livenessProbe.failureThreshold }}
```

Ya tenemos nuestro fichero de despliegue listo, ahora vamos a modificar el fichero `service.yaml` para exponer nuestra aplicación al mundo.
Un servicio permite que la aplicación reciba trafico a través de una dirección IP. Los servicios se pueden exponer de diferentes formas especificando un tipo:


|Tipo            | Descripción
|----------------|-------------------------------
|**ClusterIP**| Solo se puede acceder al servicio mediante una IP interna desde el cluster.
|**NodePort**| Se puede acceder al servicio desde fuera del clúster a través de NodeIP y NodePort
|**LoadBalancer**| Se puede acceder al servicio desde fuera del clúster a través de un equilibrador de carga externo. Puede ingresar a la aplicación.

En nuestro caso vamos a utilizar el tipo de LoadBalancer. Esto lo vamos a expecificar en el fichero `values.yaml`, ahora vamos a crear el fichero``service.yam`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "appPython3.fullname" . }}
  labels:
    app: {{ template "appPython3.name" . }}
    chart: {{ template "appPython3.chart" . }}
    release: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.externalPort }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: {{ .Values.service.externalprotocol }}
      name: {{ .Values.service.externalName }}
  selector:
    app: {{ template "appPython3.name" . }}
    release: {{ .Release.Name }}

```

Añadimos algunas label, para tener un control y una comodidad a la hora de listar y trabajar con los objetos que vamos a crear
```yaml
labels:
  app: {{ template "appPython3.name" . }}
  chart: {{ template "appPython3.chart" . }}
  release: {{ .Release.Name }}
```

```yaml
ports:
  - port: {{ .Values.service.externalPort }}
    targetPort: {{ .Values.service.targetPort }}
    protocol: {{ .Values.service.externalprotocol }}
    name: {{ .Values.service.externalName }}
```

Por útimo, vamos a definir todos los valores de las configuraciones que hemos realizado en los pasos anteriores.

```yaml
# Default values for appPython3.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: jcdemo/flaskapp
  pullPolicy: IfNotPresent
  tag: latest

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

## Service Account
## Ref: https://kubernetes.io/docs/admin/service-accounts-admin/
##
serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # If not set and create is true, a name is generated using the fullname template
  name: ""

## Configuration values for the postgresql dependency
## ref: https://github.com/kubernetes/charts/blob/master/stable/mongodb/README.md
##
postgresql:
  enabled: true
  image:
    tag: 11.10.0-debian-10-r2
    pullPolicy: IfNotPresent
  persistence:
    size: 1Gi
  Username: userpostgresql
  Password: passpostgresql
  Database: dbpostgresql


## Probes
## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
readinessProbe:
  path: /
  # port: 9000
  initialDelaySeconds: 0
  timeoutSeconds: 1
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 3

livenessProbe:
  path: /
  # port: 9000
  initialDelaySeconds: 0
  timeoutSeconds: 1
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 3

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80
  name: http
  internalPort: 3000
  internalProtocol: TCP
  externalPort: 80
  targetPort: 
  externalprotocol: TCP
  externalName: http

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
```


```yaml

```

-----------------------

Comprobamos si el chart es correctamente y se puede pasar a instalarlo.

> **debian@cliente:***~/app-python3* **$** `helm lint ./`
~~~
==> Linting ./
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
~~~

Nos pide que le añadamos el atributo ``icon``.
> **debian@cliente:***~/app-python3* **$** `nano Chart.yaml`
~~~
icon: https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png
~~~

REalizamos de nuevo la comprobación y nos muestra que esta todo correcto
> **debian@cliente:***~/app-python3* **$** `helm lint ./`
~~~
==> Linting ./

1 chart(s) linted, 0 chart(s) failed
~~~

Instalamos el chart y los listamos para comprobar su estado.
> **debian@cliente:***~* **$** ``helm install test1 app-python3/``

> **debian@cliente:***~* **$** ``helm list``

~~~
NAME    NAMESPACE   REVISION   UPDATED                                   STATU      CHART               APP VERSION
test1   default     1          2020-11-11 12:35:22.083675424 +0000 UTC   deployed   app-python3-0.1.0   1.0.0
~~~