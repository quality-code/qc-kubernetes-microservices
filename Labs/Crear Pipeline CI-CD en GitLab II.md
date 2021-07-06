
# GitLab CI/CD II

En este laboratorio, verá cómo iniciar a construir procesos CI/CD en un runner OnPremise

# Pre-requisitos

Para utilizar GitLab CI/CD:

 - Cree un proyecto en blanco en GitLab.
 - Asegúrese de tener **runners onPremise** habilitados y disponibles para ejecutar sus Jobs con ejecutores **shell y docker** configurados.

# Configuración de un Pipeline
 
En GitLab el pipeline se configura en el fichero *.gitlab-ci.yml* que tiene que estar en la raíz del repositorio.

GitLab en cuanto detecta la presencia de este fichero comienza la ejecución del pipeline definido.

En la primera parte del fichero definimos los distintos stages o fases que componen nuestro pipeline. Por ejemplo:

    stages:
      - build
      - test
      - produccion

A continuación tenemos que definir los jobs que se van a ejecutar en cada stage del pipeline, para asociar un job a su stage utilizamos la propiedad «**stage**» teniendo en cuenta que si dos jobs tienen la misma propiedad «stage» se van a ejecutar en paralelo siempre que el número de runners lo permita. En este caso definimos dos jobs: el primero que se va a ejecutar en un runner que tenga *tag docker*; y el segundo que se va a ejecutar en un runner que tenga *tag shell*.

    version:
      stage: build
      image: alpine:latest
      script:
	    - echo "Instruccion ejecutada en Runner Local de Docker"
        - whoami
      tags:
        - docker
    
    build:
      stage: test
      script:
        - docker run -d --name webserver -p 8080:80 nginx 
        - docker ps
      tags:
        - shell

Dentro del job definimos tareas a ejecutar. Estas tareas se definen bajo la propiedad «**script**» como vemos en el ejemplo de arriba. El job de tipo **docker** ejecutará el comando dentro del contenedor que venga definido en la propiedad «**image**», teniendo en cuenta que GitLab va a copiar por nosotros todo el contenido del repositorio en la raíz del contenedor. En el segundo ejemplo, las tareas las ejecutamos en la **shell** de la máquina.

Puede ser que solo nos interese ejecutar un job definido cuando estemos en una determinada rama, como en el ejemplo, ejecutar el despliegue de una release o de producción en función de si estamos en una rama de release o estamos en la rama master. Para eso utiilizamos la propiedad «**only**».

Agregue el siguiente contenido al final del archivo.

    prod-job:
      stage: produccion
      script:
        - docker run -d --name webserverprd -p 80:80 nginx
      only:
        - master
      tags:
        - shell

Realice un commit  y valide el log del pipeline para ver los resultados.

De igual forma, tambien puede abrir un navegador web en su maquina local e ingresar a la url http://localhost:8080 para ver el servidor de **nginx de pruebas** o a la url http://localhost:80 para ver el **servidor de producción**.


## Buen Trabajo
