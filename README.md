# Ejercicio: Despliegue de la aplicación _Bookinfo_ en Kubernetes

Este ejercicio está diseñado para ayudarte a practicar y comprender conceptos avanzados de networking en Kubernetes, un aspecto crucial para el examen de Certified Kubernetes Administrator (CKA). Implementarás la aplicación Bookinfo—originalmente de Istio—sin utilizar Istio. El ejercicio se centra en configurar componentes de networking como Services, Ingress y Network Policies, además de practicar afinidad de nodos, tolerancias y taints. También deberás decidir la mejor manera de desplegar los pods proporcionados (Deployment, StatefulSet u otros) y elegir los tipos de servicios adecuados.

## **Pre-requisitos:**

- Un entorno Kubernetes multinodo proporcionado por Vagrant (se te proporcionarán los scripts de Vagrant) o en su defecto minikube de 2 nodos.
- `kubectl` instalado y configurado para interactuar con tu clúster.
- Conocimientos básicos de los recursos de Kubernetes y Networking.

## **Pasos del Ejercicio:**

### Inicio: Haz un fork en tu GitHub personal

Para comenzar este ejercicio, lo primero que debes hacer es **crear un fork del repositorio original en tu cuenta personal de GitHub**. Esto te permitirá tener una copia del proyecto en la que podrás trabajar y realizar cambios sin afectar el repositorio original. Una vez creado el fork, clona el repositorio en tu máquina local para comenzar a trabajar en el ejercicio.

Esto es necesario para poder entregar el ejercicio al final.

### **Parte 1: Preparar el Entorno**

**Paso 1: Configurar el Clúster Kubernetes con Vagrant**

- Utiliza los scripts de Vagrant proporcionados para iniciar un clúster Kubernetes multinodo.

  ```bash
  vagrant up
  ```

- De forma alternativa, si tienes problemas con vagrant, puedes utilizar minikube con 2 o 3 nodos dependiendo de la capacidad de tu máquina:
  ```bash
  minikube start --nodes 2
  ```
- Verifica que los nodos estén funcionando:
  ```bash
  kubectl get nodes
  ```

---

### **Parte 2: Aplicar los deployments de BookInfo**

**Paso 1: Revisar los ficheros proporcionados**

En el directorio `kubernetes` tienes una colección de manifestos yaml con los deployment de los diferentes servicios.

En este caso la base de datos es un deployment (y no un StatefulSet) por simplicidad, ya que nos centraremos en networking en vez de volúmenes.

**Paso 2: Crear los recursos**

- Lanzar un `apply` para crear todos los deployments

**Paso 6: Verificar Implementaciones y Pods**

- Comprueba que todos los pods estén en estado `Running`:
  ```bash
  kubectl get pods
  ```

---

### **Parte 3: Configurar los Servicios**

**Paso 1: Decidir los Tipos de Servicios**

- Decide qué tipo de Service crear para cada microservicio (`ClusterIP`, `NodePort`, `LoadBalancer`, etc.).
- Justifica tu elección en base a cómo se comunicarán los servicios entre sí y con el exterior.

**Paso 2: Crear los Servicios**

- Crea los archivos YAML para los Services y aplícalos al clúster.

**Paso 3: Probar la Comunicación entre Servicios**

- Entra en un pod `productpage`
- Dentro del pod, prueba la conectividad al resto de servicios.
  ```bash
  curl <http://details:9080/details/0>
  curl <http://reviews:9080/reviews/0>
  curl <http://ratings:9080/ratings/0>
  ```

**Paso 4: Comprobar DNS**

Utiliza el comando `nslookup` o `dig` dentro de un pod para verificar que los servicios se resuelven correctamente:

```bash
nslookup details.default.svc.cluster.local
nslookup reviews.default.svc.cluster.local
nslookup ratings.default.svc.cluster.local
```

Asegúrate de que los nombres de los servicios se resuelven a las IPs correctas dentro del clúster.

---

### **Parte 4: Practicar Afinidad de Nodos, Tolerancias y Taints**

**Paso 1: Añadir Taints a los Nodos**

- Añade un taint a uno de los nodos del clúster:
  ```bash
  kubectl taint nodes <node-name> key=value:NoSchedule
  ```
- Verifica que el taint se ha aplicado correctamente:

  ```bash
  kubectl describe node <node-name>
  ```

- Haz un rollout restart en los deployments y comprueba como los pods no se programan en el nodo tainteado.

  ```bash
  kubectl rollout restart deployment <deployment-name>
  ```

  ```bash
  kubectl get pods -o wide
  ```

**Paso 2: Configurar Tolerancias en los Pods**

- Modifica las definiciones de los pods o Deployments para incluir una tolerancia que permita programarlos en el nodo tainted.

  ```yaml
  tolerations:
    - key: "key"
      operator: "Equal"
      value: "value"
      effect: "NoSchedule"
  ```

- Aplica los cambios y verifica que los pods se programan en el nodo tainteado.

**Paso 3: Configurar Afinidad de Nodos**

- Añade reglas de afinidad de nodos para que ciertos pods se ejecuten preferentemente en nodos específicos.
  ```yaml
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: <node-label-key>
                operator: In
                values:
                  - <node-label-value>
  ```

**Paso 4: Aplicar los Cambios**

- Aplica las modificaciones a tu clúster y verifica en qué nodos se programan los pods:
  ```bash
  kubectl get pods -o wide
  ```

---

### **Parte 5: Exponer la Aplicación Externamente con Ingress**

**Paso 1: Instalar un Controlador Ingress**

- Si no está instalado, despliega un controlador Ingress NGINX:
  ```bash
  kubectl apply -f <https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml>
  ```

**Paso 2: Crear un Recurso Ingress**

- Crea un archivo llamado yaml para el recurso ingress. Configúralo para responder al host `bookinfo.com` y el path `/` en modo `Prefix` debe apuntar al servicio `productpage` en su puerto `9080`.

**Paso 3: Actualizar el Archivo Hosts Local**

- Mapea `bookinfo.com` a la IP externa del controlador Ingress.
  - Encuentra la IP:
    ```bash
    kubectl get svc -n ingress-nginx
    ```
  - Edita `/etc/hosts` en tu máquina de forma temporal, así tendremos acceso desde nuestro host al ingress en cuesti´øn.:
    ```
    <INGRESS_IP> bookinfo.example.com
    ```

**Paso 4: Acceder a la Aplicación**

- Abre `http://bookinfo.com` en tu navegador para verificar el acceso externo.

---

### **Parte 6: Implementar Network Policies**

**Paso 1: Crear Espacios de Nombres Separados**

- Crea namespaces para separar los componentes:
  ```bash
  kubectl create namespace frontend
  kubectl create namespace backend
  ```

**Paso 2: Asignar Recursos a los Namespaces**

- Implementa `productpage` en el namespace `frontend`.
- Implementa `details`, `reviews` y `ratings` en el namespace `backend`.

**Paso 3: Etiquetar los Namespaces**

- Añade etiquetas a los namespaces para facilitar las políticas:
  ```bash
  kubectl label namespace frontend role=frontend
  kubectl label namespace backend role=backend

  ```

**Paso 4: Definir Network Policies**

- Crea una política para denegar todo el tráfico de entrada en `backend`.

**Paso 5: Permitir Tráfico Específico**

- Crea una política para permitir tráfico en el namespace `backend` desde el namespace `frontend`.

**Paso 6: Probar las Network Policies**

- Intenta acceder a los servicios de backend desde un pod en `frontend` (debería ser permitido).
- Intenta acceder desde un pod en el namespace `default` (debería ser denegado).

---

**Entregables:**

Cuando termines, crea un _Pull Request_ con tu nombre e apellido de titulo hacia el repositorio original.

**Conclusión:**

Este ejercicio proporciona experiencia práctica con los componentes de networking de Kubernetes, esenciales para el examen CKA. Al desplegar una aplicación de microservicios y configurar recursos de networking, además de practicar afinidad de nodos, tolerancias y taints, adquirirás habilidades prácticas en la gestión y solución de problemas en el networking de Kubernetes.
