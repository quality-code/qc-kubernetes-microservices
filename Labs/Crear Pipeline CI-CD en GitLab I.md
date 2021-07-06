
# GitLab CI/CD

En este laboratorio, verá cómo iniciar a construir procesos CI/CD.

# Pre-requisitos

Para utilizar GitLab CI/CD:

 - Cree un proyecto en blanco en GitLab.
 - Asegúrese de tener **runners compartidos** habilitados y disponibles para ejecutar sus Jobs.

# Crear Manifiesto CI
 
1. Para crear un archivo `.gitlab-ci.yml`, en el menu lateral, seleccione **CI/CD > Editor**.
2. De click en el botón **Create new CI/CD pipeline**.
3. Borre todo el contenido del archivo y agregue el siguiente contenido para configurar los **Stage** a trabajar dentro del pipeline.

```
stages:
  - compilacion
  - pruebas
  - despliegue
```
4. Agregue la siguiente configuracion para crear **4 Jobs** y cada uno asociado a un **stage**:
```
    dev-job:
        stage: compilacion
            
    selenium-job:
        stage: pruebas

    soapui-job:
        stage: pruebas

	prod-job:
        stage: despliegue
```

5. Justo abajo de la propiedad **stage**, agregue la siguiente configuración para cada job:
```
	# Script dev-job
    script:
        - echo "Aqui se ejecutarán todas las tareas para compilar nuestro proyecto"
```
```
	# Script selenium-job
    script:
        - echo "Aqui se ejecutarán todas las pruebas funcionales con Selenium"
        - sleep 15 # Se daran 15 segundos para simular la ejecución de las pruebas
```
```
	# Script soapui-job
    script:
        - echo "Aqui se ejecutarán todas las pruebas a las APIs del proyecto"
        - sleep 10 # Se daran 10 segundos para simular la ejecución de las pruebas
```
```
	# Script prod-job
    script:
        - echo "Finalizadas las Pruebas, se realizara el Despliegue en Produccion"
        - sleep 13 # Se daran 13 segundos para simular el paso a producción
```
# Creación de Variables

Las variables se pueden definir a nivel de proyecto, pipeline o job, en este ejercicio los configurará a nivel de pipeline. Agregue el siguiente contenido al inicio del archivo.
```
variables:
    ServerDb: "myserver.database.net"
    UserDb: "admindb"
```
Agregue un script mas dentro del job **dev-job** justo abajo del script `echo "Aqui se ejecutarán todas las tareas para compilar nuestro proyecto"`:

    - echo "Conectado al Server $ServerDb con el Usuario $UserDb"
Haga commit dando click en el botón **Commit changes** el pipeline iniciará automáticamente cuando el commit haya finalizado.

# Ver Status de su Pipeline y Jobs

Cuando finalice el commit, iniciara el pipeline.

Para ver su pipeline, vaya a **CI/CD > Pipelines** deme mostrarle un pipeline con 4 stages:

![Pipeline](https://github.com/quality-code/qc-kubernetes-microservices/blob/main/Resources/PipelineGitLab.png)

Para ver una representación viual de su pipeline, haga click en el ID del Pipeline.

![Pipeline Status](https://github.com/quality-code/qc-kubernetes-microservices/blob/main/Resources/PipelineGitLab2.png)

Para ver detalles de un Job, haga click en el nombre del Job, por ejemplo, `dev-job`.

![Pipeline Details](https://github.com/quality-code/qc-kubernetes-microservices/blob/main/Resources/PipelineGitLab3.png)

El Job finalizó exitosamente dentro de un runner compartido. Asegúrese de que todos los demas Jobs hayan finalizado correctamente.

## Buen Trabajo
