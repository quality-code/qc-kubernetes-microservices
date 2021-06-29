# Horizontal Pod Autoscaler(HPA) basado en CPU y Memoria

![HPA Architecture](https://github.com/quality-code/qc-kubernetes-microservices/blob/main/Resources/Kube1.png)

En este laboratorio, verá cómo puede escalar los pods de Kubernetes utilizando el escalador automático horizontal de pods (HPA) en función de la CPU y la memoria.

La imagen a trabajar es una implementación de **php** de ejemplo que tiene algunas tareas básicas de procesamiento intensivo.

# Crear Manifiesto YAML

Cree un archivo llamado **qc-php-deployment.yaml** con el siguiente contenido:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: php-apache
    spec:
      selector:
        matchLabels:
          run: php-apache
      replicas: 1
      template:
        metadata:
          labels:
            run: php-apache
        spec:
          containers:
          - name: php-apache
            image: qualitycode1/php-hpa
            ports:
            - containerPort: 80
            resources:
              limits:
                cpu: 500m
                memory: 1Gi
              requests:
                cpu: 200m
                memory: 0.5Gi
    
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: php-apache
      labels:
        run: php-apache
    spec:
      ports:
      - port: 80
      selector:
        run: php-apache

# Desplegar manifiesto

Ejecute la siguiente instrucción para desplegar el contenedor en Kubernetes:

    kubectl apply -f qc-php-deployment.yaml

Consulte los pods una vez finalizado el despliegue:

    kubectl get pods

# Crear HPA con utilización de CPU

Cree un archivo llamado qc-php-deployment-hpa-cpu.yaml y agregue el siguiente contenido para crear la regla de auto-escalado HPA:

    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
      name: php-apache
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: php-apache
      minReplicas: 1
      maxReplicas: 10
      targetCPUUtilizationPercentage: 50

Para el HPA, estamos definiendo el escalado automático en función de la utilización de CPU del 50% y las réplicas máximas en 10 pods y como mínimo en 1 pod. Los pods se reducirán después de un período de estabilización de 1 m.

Ejecute el manifiesto:

    kubectl apply -f qc-php-deployment-hpa-cpu.yaml

# Generar Carga

Ejecute un contenedor para lanzar desde allí carga al Pod de la aplicación de PHP:

kubectl run -it --rm load-generator --image busybox /bin/sh

una vez se encuentre en el Shell del Pod, ejecute la siguiente instrucción para iniciar la carga:

while true; do wget -q -O-_http://php-apache; done
Aparecerá un mensaje como el siguiente:

    while true; do wget -q -O- http://php-apache; done
    OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
    OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!

Verifique el HPA y el Deployment

    kubectl get hpa

El resultado será como el siguiente:

    NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    php-apache   Deployment/php-apache   86%/50%   1         10        4          15m

Pods escalados automáticamente según la utilización de la CPU de 1 a  4 y está en modo creciente.

    NAME                         READY   STATUS    RESTARTS   AGE
    load-generator               1/1     Running   0          12m
    php-apache-d5bd8446b-277m5   1/1     Running   0          42m
    php-apache-d5bd8446b-6hhcj   0/1     Pending   0          7m39s
    php-apache-d5bd8446b-blwf2   1/1     Running   0          7m55s
    php-apache-d5bd8446b-f7486   0/1     Pending   0          6m54s
    php-apache-d5bd8446b-hgwzw   0/1     Pending   0          7m39s


# Detener la geneeración de carga

En el **shell** donde se está ejecutando la carga hacia el pod, ejecute las teclas **ctrl + c**.

Todos los pods se redujeron a 1 y la utilización de la CPU también se redujo.

Consulte nuevamente los pods:

    kubectl get pods

Si los pods no han disminuido, espere unos minutos y vuelva a ejecutar la instrucción.

    NAME                         READY   STATUS    RESTARTS   AGE
    php-apache-d5bd8446b-277m5   1/1     Running   0          51m

# Crear un nuevo HPA basado en la utilización de Memoria

Cree un HPA llamado **qc-php-deployment-hpa-memory.yaml**, mencione el valor de avgUtilization como 10Mi, ya que cada pod se está ejecutando actualmente a 12-13 Mi.

    apiVersion: autoscaling/v2beta2 
    kind: HorizontalPodAutoscaler
    metadata:
      name: php-memory-scale 
    spec:
      scaleTargetRef:
        apiVersion: apps/v1 
        kind: Deployment 
        name: php-apache 
      minReplicas: 1 
      maxReplicas: 10 
      metrics: 
      - type: Resource
        resource:
          name: memory 
          target:
            type: Utilization 
            averageValue: 10Mi 

Valide el consumo actual:

    kubectl top pod

    NAME                         CPU(cores)   MEMORY(bytes)
    php-apache-d5bd8446b-277m5   1m           12Mi

Aplique el nuevo HPA:

    kubectl apply -f qc-php-deployment-hpa-memory.yaml

Revise el estado y cantidad de los Pods:

    kubectl get pods
    
    NAME                         READY   STATUS    RESTARTS   AGE
    php-apache-d5bd8446b-277m5   1/1     Running   0          64m
    php-apache-d5bd8446b-zt2v4   1/1     Running   0          29s

 Ahora el HPA

    kubectl get hpa

    NAME               REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
    php-apache         Deployment/php-apache   0%/50%          1         10        2          43m
    php-memory-scale   Deployment/php-apache   13590528/10Mi   1         10        2          34s


## Buen Trabajo
