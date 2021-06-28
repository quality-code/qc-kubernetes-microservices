# Configurar cuotas de memoria y CPU para un espacio de nombres

En este laboratorio se muestra cómo establecer cuotas para la cantidad total de memoria y CPU que pueden usar todos los contenedores que se ejecutan en un espacio de nombres. Las cuotas se especifican en un objeto [ResourceQuota](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#resourcequota-v1-core)

# Pre-requisitos

Debe tener un clúster de **Kubernetes** y la herramienta de línea de comandos **kubectl** debe configurarse para comunicarse con el clúster. Se recomienda ejecutar este tutorial en un clúster con al menos dos nodos que no actúan como hosts del plano de control.

Para comprobar la versión, escriba. `kubectl version`

Cada nodo del clúster debe tener al menos 1 GiB de memoria.

# Crear un espacio de nombres

Cree un espacio de nombres para que los recursos que cree en este ejercicio se aíslen del resto del clúster.

```
kubectl create namespace qc-ns-quota-mem-cpu
```
# Crear un ResourceQuota

Cree un archivo llamado **quota-mem-cpu.yaml**, y agregue el siguiente contenido para un objeto ResourceQuota:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-mem-cpu
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Ejecute la siguiente instrucción para crear el *ResourceQuota*:

```shell
kubectl apply -f quota-mem-cpu.yaml --namespace qc-ns-quota-mem-cpu
```
Para ver la información detallada sobre el *ResourceQuota*:

```shell
kubectl get resourcequota quota-mem-cpu --namespace qc-ns-quota-mem-cpu --output yaml
```
*ResourceQuota* coloca estos requisitos en el espacio de nombres **qc-ns-quota-mem-cpu**:

-   Cada contenedor debe tener una solicitud de memoria, un límite de memoria, una solicitud de CPU y un límite de CPU.
-   El total de solicitudes de memoria para todos los contenedores no debe superar 1 GiB.
-   El límite de memoria total para todos los contenedores no debe superar los 2 GiB.
-   El total de solicitudes de CPU para todos los contenedores no debe superar 1 CPU.
-   El total del límite de CPU para todos los contenedores no debe superar los 2 cpu.

# Crear primer Pod

Cree un archivo llamado **qc-pod-quota1.yaml** y agregue el siguiente contenido:

    apiVersion: v1
    kind: Pod
    metadata:
      name: qc-pod-quota1
    spec:
      containers:
      - name: qc-pod-quota1
        image: nginx
        resources:
          limits:
            memory: "800Mi"
            cpu: "800m"
          requests:
            memory: "600Mi"
            cpu: "400m"

Para inicializar el Pod, ejecute la siguiente línea:

    kubectl apply -f qc-pod-quota1.yaml --namespace qc-ns-quota-mem-cpu

Compruebe que el contenedor del pod se está ejecutando:

```
kubectl get pod qc-pod-quota1 --namespace qc-ns-quota-mem-cpu
```

Una vez más, vea la información detallada sobre *ResourceQuota*:

```
kubectl get resourcequota quota-mem-cpu --namespace qc-ns-quota-mem-cpu --output yaml
```

El resultado muestra la cuota junto con la cantidad de la cuota que se ha utilizado. Puede ver que las solicitudes y límites de memoria y CPU para su Pod no superan la cuota.

```
status:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
  used:
    limits.cpu: 800m
    limits.memory: 800Mi
    requests.cpu: 400m
    requests.memory: 600Mi
```

# Crear segundo Pod

Cree otro archivo con el nombre **qc-pod-quota2.yaml** y agregue la siguiente configuración:

    apiVersion: v1
    kind: Pod
    metadata:
      name: qc-pod-quota2
    spec:
      containers:
      - name: qc-pod-quota2
        image: redis
        resources:
          limits:
            memory: "1Gi"
            cpu: "800m"
          requests:
            memory: "700Mi"
            cpu: "400m"

En el archivo de configuración, usted puede ver que el Pod tiene una petición de memoria del MiB 700. Observe que la suma de la solicitud de memoria utilizada y esta nueva solicitud de memoria supera la cuota de solicitud de memoria. 600 MiB + 700 MiB > 1 GiB.

Intente crear el Pod:

```shell
kubectl apply -f qc-pod-quota2.yaml --namespace qc-ns-quota-mem-cpu
```

El segundo Pod no se crea. El resultado muestra que la creación del segundo pod haría que el total de la solicitud de memoria superara la cuota de solicitud de memoria.

```
Error from server (Forbidden): error when creating "qc-pod-quota2.yaml":
pods "qc-pod-quota2" is forbidden: exceeded quota: quota-mem-cpu, 
requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi
```

# Conclusión

Como ha visto en este laboratorio, puede usar *ResourceQuota* para restringir el total de solicitudes de memoria para todos los contenedores que se ejecutan en un espacio de nombres. También puede restringir los totales para el límite de memoria, la solicitud de CPU y el límite de CPU.

## Buen Trabajo :)
