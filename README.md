# <h1 align=center> Tekton Kubernetes Angular CRUD

Aplicación Web desarrollada en el Stack MEAN para realizar las operaciones CRUD integrada con la herramienta Tekton para el despliegue e integración automáticos en un cluster de Kubernetes.

Esta demo es un acercamiento inicial que permite crear, leer, actualizar y borrar registros de una base de datos por medio de llamados al API de Express que está conectado con la base de datos MongoDB.

Con Tekton aseguramos el despliegue en un cluster de Kubernetes con la opción de realizar cambios que se vean reflejados en el entorno de producción en el menor tiempo posible con las respectivas pruebas, facilitando el Test-Driven Development (TDD) y las metodologías ágiles como DevOps.

El repositorio se encuentra organizado en dos submodulos, Tekton_Back y Tekton_Front, cada uno con la respectiva parte de la aplicación.

## :package: Arquitectura

![arquitectura](https://link)

## Requisitos

- Tener un servicio de **[Kubernetes Cluster]()** disponible en la cuenta IBM Cloud. Para cuentas Lite este servicio está disponible por 30 días.
- :cloud: [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started&locale=en)
- :whale: [Docker](https://www.docker.com/products/docker-desktop)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/). La version de esta herramienta debe ser compatible con la version de IKS que se desplegó en la cuenta.
- [TypeScript](https://www.typescriptlang.org/#download-links)
- [Angular CLI](https://cli.angular.io/)
- [Node y NPM](https://nodejs.org/en/)

## :hand: Hands On!

### Instalar o actualizar los plugins necesarios del IBM Cloud CLI.

Reemplazar `install` con `update` si es el caso:

```sh
ibmcloud plugin install kubernetes-service
ibmcloud plugin install container-registry
```

### Desplegar la imagen de nuestra base de datos.

Para este caso vamos a desplegar la base de datos en un pod separado a nuestra aplicación para tener mayor control sobre ella y su aseguramiento, por lo que no usaremos Tekton para esta imagen sino que lo haremos de forma manual. Con este fin, necesitaremos configurar nuestra CLI.

- Iniciar sesión en nuestro terminal de comandos:

  `ibmcloud cr login`

  **Importante**: Tener en cuenta la región que aparece en pantalla, ya que tomará parte de los pasos siguientes.

- Crear espacio de nombres:

  `ibmcloud cr namespace-add <namespace>`

  Este espacio de nombres tambien formará parte de los comandos siguientes. Se puede comprobar el nombre en cualquier momento usando el comando: `ibmcloud cr namespaces`

- Descargar la imagen de MongoDB del registro Docker Hub:<br/>
  Para descargar la imagen oficial de MongoDB, almacenada en el registro de imagenes Docker Hub, utilizar el comando:

  `docker pull mongo:latest`

- Cambiar tag de la imagen para que sea compatible con la region y el espacio de nombre de nuestro IKS:<br/>
  `docker tag mongo:latest <region>/<namespace>/mongo`

- Subir imagen a Container Registery:
  `docker push <region>/<namespace>/mongo`
- Configurar el plugin kubernetes-service:

  `ibmcloud cs cluster config --cluster <nombre_cluster>`

  Si no conocemos el nombre de nuestro cluster podemos utilizar el comando `ibmcloud cs clusters` y seleccionar el que vayamos a utilizar

- Desplegar la imagen en nuestro Cluster:

  `kubectl run mongo --image=<region>/<namespace>/mongo`

- Exponer el Pod creado:

  `kubectl expose deployment/mongo --type="NodePort" --port=27017`

  Para poder enlazar la imagen creada con nuestra aplicación necesitamos la IP y el puerto generado en el paso anterior, para eso ejecutamos el comando:

- Puerto: `kubectl describe service mongo`

- IP: `ibmcloud cs workers --cluster <nombre_cluster>`

  Con la IP y el puerto editamos el archivo database.ts ubicado en la carpeta **Tekton_Back/src**, modificando las lineas de conexión con la base de datos, como en la siguiente imagen:

  ![base de datos](.github\base_de_datos.png)

### Desplegar nuestra aplicación con Tekton

<p align=center><img src=".github\tekton-pipelines.png" style="width: 48px;">


Para desplegar nuestra aplicación de Backend, nos dirigimos al repositorio Tekton_Back, que se encuentra enlazado como submodulo, donde podemos encontrar la carpeta .tekton, la carpeta scripts, el archivo deployment.yml y el archivo Dockerfile, recursos que utilizará Tekton para realizar el despliegue.