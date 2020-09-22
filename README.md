# <h1 align=center> Hands On: Despliegue de una aplicación usando Tekton en IKS

Aplicación Web desarrollada en Angular para crear listas de objetos, integrada con la herramienta Tekton para el despliegue e integración automáticos en un cluster de Kubernetes.

Con Tekton aseguramos el despliegue en un cluster de Kubernetes con la opción de realizar cambios que se vean reflejados en el entorno de producción en el menor tiempo posible con las respectivas pruebas, facilitando el Test-Driven Development (TDD) y las metodologías ágiles como DevOps.

## 📑 Tabla de contenido

1. [Requisitos](#-requisitos)
2. [Hands On!](#-hands-on)<br>
   2.1 [Depliegue de la aplicación con Tekton](#-desplegar-nuestra-aplicación-con-tekton)<br>
3. [Extras: Private Worker](#extras-private-worker)
4. [Recursos Adicionales](#recursos-adicionales)

## Requisitos

**Nota:** Los requisitos de CLI, marcados con un \*, no son necesarios en caso de usar [IBM Cloud Shell](https://cloud.ibm.com/shell)

- Tener un servicio de **[Kubernetes Cluster](https://cloud.ibm.com/kubernetes/clusters)** disponible en la cuenta IBM Cloud. Para cuentas Lite este servicio está disponible por 30 días.
- :cloud: [IBM Cloud CLI \*](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started&locale=en)
- :whale: [Docker \*](https://www.docker.com/products/docker-desktop)
- [kubectl \*](https://kubernetes.io/docs/tasks/tools/install-kubectl/). La version de esta herramienta debe ser compatible con la version de IKS que se desplegó en la cuenta.

## :hand: Hands On!

### <img src=".github/tekton-pipelines.png" height="48"> Desplegar nuestra aplicación con Tekton

El repositorio ya cuenta con los archivos de configuración necesarios, la carpeta .tekton, la carpeta scripts, el archivo deployment.yml y el archivo Dockerfile, recursos que utilizará Tekton para realizar el despliegue.

<p align=center><img src=".github/tekton-files.png" height="400"></p>

1. Nos dirigimos a la página de [IBM Cloud ☁️](https://cloud.ibm.com), iniciamos sesión y damos clic a la sección de DevOps en el panel lateral izquierdo, como se ve en la imagen.

<p align=center><img src=".github/devops-section.png" height="450"></p>

2. Procedemos a crear un nuevo toolchain (cadena de herramientas) seleccionando el botón create toolchain.

**Importante**: Revisar que la localización de nuestra toolchain sea la misma a nuestro cluster de Kubernetes, para esta demo usaremos **Washington DC** y el grupo de recursos **openshift-workshop**.

En la pantalla de creación seleccionamos el recuadro Build your own toolchain en la sección de Other Templates.

<p align=center><img src=".github/devops-create.png" height="450"></p>

3. Escogemos el nombre de nuestra toolchain, la región de nuestro cluster y crear.

   Nos debe redirigir a la siguiente pantalla.

<p align=center><img src=".github/devops-home.png"></p>

4. Añadimos las siguientes tools, con la configuración mostrada en las imágenes:

- GitHub. Repositorio utilizado`https://github.com/emeloibmco/ibm-Kubernetes-Service-Tekton`

 <p align=center><img src=".github/devops-github.png"></p>

- Delivery Pipeline.

Escoger en tipo de pipeline, Tekton

 <p align=center><img src=".github/devops-pipe.png"></p>

Así deberá quedar el Toolchain:

<p align=center><img src=".github/devops-tool.png"></p>

**Nota:** No es necesario que aparezca el cuadro Delivery Pipeline Private Worker ya que este aplica para la sección adicional al final del documento

Ahora debemos enlazar el repositorio de la aplicación y el Worker que vamos a usar, con el Pipeline creado en el paso anterior. Para eso ingresamos a nuestro Delivery Pipeline.

### Definitions

En el menú Definitions agregamos nuestro repositorio, como ya se encuentra enlazado nos aparecerá como única opción, y nos reconocerá la carpeta .tekton.

### <p align=center><img src=".github/devops-gitTek.png"></p>

Una vez añadido deberá aparecer la definición de nuestro repositorio y guardamos los cambios.

Ahora vamos a la pestaña de Worker, donde seleccionaremos el worker público que provee IBM Cloud.

**Importante: para utilizar un Private Worker encontrará los pasos en la sección dedicada al final del documento**

### <p align=center><img src=".github/devops-tekworker.png"></p>

Guardamos los cambios.

### Triggers

Acá configuraremos la forma en la que nuestro pipeline va a iniciar. En Add Trigger tenemos cuatro opciones, para este Hands On crearemos Git Repository y Manual.

- Git Repository: Seleccionamos nuestro repositorio, la rama que queremos desplegar y las acciones que ejecutarán nuestro pipeline. En la siguiente imagen se muestra la configuración para esta guía.

### <p align=center><img src=".github/devops-gitTrigger.png"></p>

- Manual Trigger: habilitaremos un trigger manual para poder ejecutar nuestro pipeline aunque no tengamos cambios nuevos en nuestra aplicación.

### <p align=center><img src=".github/devops-manTrigger.png"></p>

Guardamos los cambios.

### Variables de Entorno

Añadimos 4 propiedades de Texto y una propiedad segura, como se muestra en la imagen, asignando los valores según la configuración de nuestro cluster y una llave API.

- cluster: El nombre del cluster sobre el que vamos a realizar el despliegue, en este caso, k8s-cluster.
- clusterNamespace: El espacio de nombres donde se almacenarán los recursos k8s creados. Utilizar el formato `<su-nombre>-ns`.
- clusterRegion: La región en la que se encuentra nuestro cluster, en este caso, us-south.
- registryNamespace: Espacio de nombres del IBM Container Registry donde se guardará la imagen docker creada por Tekton. Ya hemos preparado un namespace `tekton-handson`.

Para crear la llave API nos dirigimos a [IBM Cloud API keys](https://cloud.ibm.com/iam/apikeys?cm_mmc=IBMBluemixGarageMethod-_-MethodSite-_-10-19-15::12-31-18-_-api-keys) y presionamos crear una IBM Cloud API key, asignando un nombre y una descripción.

**Importante**: Guardar el valor de nuestra API key, descargando el archivo.

![env](.github/devops-envVars.png)

Una vez finalizado esto podemos ir a la pestaña PipelineRuns y ejecutar manualmente un nuevo pipelineRun.

Finalmente podemos ir a nuestro Pipeline y ejecutar manualmente un nuevo pipelineRun.

### <p align=center><img src=".github/devops-success.png"></p>

---

## Extras: Private Worker

Para utilizar un private worker en lugar del público ofrecido por IBM, necesitamos generar una Service ID API Key la cual debemos guardar en el servicio de [IBM Key Protect](https://cloud.ibm.com/docs/ContinuousDelivery?topic=ContinuousDelivery-integrations#keyprotect) para poder usarla más adelante.

Debemos añadir una nueva tool, Delivery Pipeline Private Worker. Colocamos nuestra ID API Key y el nombre que deseemos y damos clic en crear herramienta.

### <p align=center><img src=".github/devops-worker.png"></p>

En el dashboard de Tekton nos aparecerá un error.

### <p align=center><img src=".github/devops-workerError.png"></p>

Esto ocurre porque no hemos enlazado el worker a nuestro Cluster de K8s. Para realizar esta integración vamos a la pantalla principal de nuestro toolchain y selecionamos nuestro worker privado.

Donde podremos ver que no hay ningún Worker en nuestra Worker Pool.

Para cambiar esto vamos al apartado de Getting Started donde están todos los pasos para añadir nuestro Worker privado, necesitaremos el API key generado anteriormente, si no lo tiene puede generar otro API key y utilizar el nuevo.

Nos aparecerán los comandos que debemos utilizar para añadir, remover o actualizar un Worker privado en nuestro cluster. **Importante**: para eso debemos ingresar a nuestro cluster por el CLI de IBM Cloud. Acá una guía de como hacerlo: [Ingresar a nuestro cluster K8s](https://cloud.ibm.com/docs/containers?topic=containers-cs_cluster_tutorial#cs_cluster_tutorial_lesson1)

### <p align=center><img src=".github/devops-kubeWorker.png"></p>

Así debera aparecer nuestra Worker Pool una vez configurado nuestro Worker

### <p align=center><img src=".github/devops-kubeWorkerConf.png"></p>

Ya con esto podemos ir al Dashboard de Tekton y ejecutar manualmente un pipelineRun.

## Recursos Adicionales

<img src=".github/tekton-pipelines.png" height="24"> [Página Oficial de Tekton](https://tekton.dev/)

:cloud: [Tekton en IBM Cloud](https://www.ibm.com/cloud/tekton)

- Si tienes preguntas o comentarios, puedes unirte en Slack al canal [IBM Cloud DevOps](https://ic-devops-slack-invite.us-south.devops.cloud.ibm.com/)

## Autores

Equipo IBM Cloud Tech Sales Colombia

**Copyright 2020 IBM**
