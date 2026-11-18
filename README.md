# Grafana en Kubernetes con Deployment y ReplicaSet

Despliegue de **Grafana** (plataforma de visualización y monitoreo) en un clúster local de **minikube**, aplicando los conceptos de **Deployment** y **ReplicaSet**.

## Contenido del repositorio

| Archivo | Recurso | Descripción |
|---|---|---|
| `grafana-deployment.yml` | Deployment | 3 réplicas de Grafana con actualización progresiva (RollingUpdate), probes de salud y límites de recursos |
| `grafana-service.yml` | Service (NodePort) | Expone Grafana en el puerto `30300` del nodo y reparte el tráfico entre los Pods |
| `grafana-replicaset.yml` | ReplicaSet | ReplicaSet independiente con 2 réplicas, para estudiar el concepto por separado |

## Conceptos

### Pod
La unidad mínima de Kubernetes: uno o varios contenedores que comparten red y almacenamiento. Si un Pod creado a mano se elimina, nadie lo vuelve a crear.

### ReplicaSet
Controlador que garantiza que siempre exista un **número fijo de Pods idénticos**. Si un Pod falla o se elimina, el ReplicaSet crea otro automáticamente (autorreparación). Identifica sus Pods mediante etiquetas (`selector.matchLabels`).

### Deployment
Nivel superior al ReplicaSet. El Deployment **crea y gestiona ReplicaSets** y añade:
- **Actualizaciones progresivas (rolling update):** cambia la versión de la aplicación Pod a Pod, sin cortar el servicio.
- **Historial y rollback:** guarda los ReplicaSets anteriores y permite volver a una versión previa.
- **Escalado declarativo:** cambiar el número de réplicas con un comando o editando el YAML.

Relación jerárquica:

```
Deployment (grafana-deployment)
   └── ReplicaSet (grafana-deployment-xxxxx)
          ├── Pod grafana-deployment-xxxxx-aaaaa
          ├── Pod grafana-deployment-xxxxx-bbbbb
          └── Pod grafana-deployment-xxxxx-ccccc
```

## Usos

**Grafana**
- Paneles (dashboards) de monitoreo de servidores, contenedores y clústeres Kubernetes junto a Prometheus.
- Visualización de métricas de negocio desde bases de datos (MySQL, PostgreSQL, InfluxDB, etc.).
- Alertas cuando una métrica supera un umbral (CPU, memoria, errores, latencia).

**Deployment**
- Desplegar aplicaciones sin estado (APIs, frontends, herramientas web como Grafana).
- Publicar nuevas versiones sin tiempo de inactividad y volver atrás si algo falla.
- Escalar la aplicación según la demanda.

**ReplicaSet**
- Asegurar alta disponibilidad: si un Pod o un nodo falla, se reemplaza automáticamente.
- Repartir la carga entre varias copias de la aplicación.
- En la práctica se usa casi siempre a través de un Deployment; usarlo solo tiene sentido cuando no se necesitan actualizaciones ni rollback.

## Requisitos
- minikube y kubectl instalados
- Docker Desktop (driver de minikube)

## Cómo desplegarlo

```bash
minikube start
kubectl apply -f grafana-deployment.yml
kubectl apply -f grafana-service.yml
kubectl apply -f grafana-replicaset.yml
kubectl get deployments,replicasets,pods,services
```

Abrir Grafana en el navegador:

```bash
minikube service grafana-service
```

Usuario `admin`, contraseña `admin123` (solo para este entorno de pruebas).

## Pruebas realizadas

**1. Autorreparación (ReplicaSet):** se elimina un Pod y el ReplicaSet crea uno nuevo.
```bash
kubectl delete pod <nombre-de-un-pod-grafana>
kubectl get pods -l app=grafana
```

**2. Escalado (Deployment):** de 3 a 5 réplicas.
```bash
kubectl scale deployment grafana-deployment --replicas=5
kubectl get pods -l app=grafana
```

**3. Actualización progresiva (Deployment):** cambio de versión de la imagen; se crea un ReplicaSet nuevo y el antiguo queda en 0.
```bash
kubectl set image deployment/grafana-deployment grafana=grafana/grafana:11.4.0
kubectl rollout status deployment/grafana-deployment
kubectl get replicasets -l app=grafana
```

**4. Rollback (Deployment):** volver a la versión anterior.
```bash
kubectl rollout history deployment/grafana-deployment
kubectl rollout undo deployment/grafana-deployment
kubectl get replicasets -l app=grafana
```

## Limpieza

```bash
kubectl delete -f .
```
