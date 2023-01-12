# Despliegues de microservicios en k8s con Helm

[![Portada](image/Portada.png)](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#despliegues-de-microservicios-en-k8s-con-helm-y-openshift)

**Proyecto de fin de curso de ASIR**

Nos vamos a centrar en el funcionamiento de Helm. Es una herramienta que gestiona recursos empaquetados y preconfigurados de Kubernetes, es decir, esta permite utilizar unas plantillas (llamadas charts) para la instalación de objetos de Kubernetes. Esta herramienta es muy útil ya que no se necesita conocimientos avanzados de Kubernetes.

**Tecnologías que se van a utilizar**

* Docker
* Kubernetes
* Helm

**Resultados que se esperan obtener** 

* Conocer el funcionamiento de Helm (En la nueva versión 3.1.0) y muchas de sus opciones.
* Realizar actualizaciones de versiones y rollback.
* Crear desde cero un chart.
* Instalar un repositorio público para alojar tus chart.

## Índice

Despliegues de microservicios en k8s con Helm

* [Introducción](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#introduci%C3%B3n)
* [Instalación de Helm v3](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#instalaci%C3%B3n-de-helm)
* [Guía rápida](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#gu%C3%ADa-r%C3%A1pida)
  * [Instalando un Chart oficial](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#instalando-un-chart-oficial)
  * [Actualizando un versión](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#actualizando-un-versi%C3%B3n)
  * [Desinstalando una versión](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#desinstalando-una-versi%C3%B3n)
  * [Realizar un Rollback](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#realizar-un-rollback)
* [Creando Charts en Helm](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#creando-charts-en-helm)
  * [Despliegue de Chart CRUD utilizando express.js y mongodb](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#despliegue-de-chart-crud-utilizando-expressjs-y-mongodb)
  * [Despliegue de Chart de aplicación PHP utilizando Laravel y MySQL](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#despliegue-de-chart-de-aplicaci%C3%B3n-php-utilizando-laravel-y-mysql)
* [Configuración de un repositorio público](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#creaci%C3%B3n-de-un-repositorio-p%C3%BAblico)
* [Siguientes pasos](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#siguientes-pasos)
* [Conclusiones](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#conclusiones)
* [Webgrafía](https://github.com/MoralG/Despliegues_de_microservicios_en_k8s_con_Helm/blob/master/Proyecto.md#webgraf%C3%ADa)
