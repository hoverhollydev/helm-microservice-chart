# Helm Microservice Chart Doc

Chart de Helm genérico para desplegar microservicios en Kubernetes.

## Descripción

Este chart despliega los siguientes recursos de Kubernetes:

- **Deployment** - Despliega el contenedor del microservicio con probes de salud
- **Service** - Expone el microservicio internamente en el cluster
- **ConfigMap** - Variables de entorno no sensibles
- **Secret** - Variables de entorno sensibles (base64)
- **HorizontalPodAutoscaler** - Escalado automático basado en CPU y memoria

## Requisitos

- Kubernetes 1.19+
- Helm 3.0+

## Instalación

### Desde repositorio local

```bash
helm install <release-name> ./helm-microservice-chart -f values.yaml
```

### Ejemplo básico

```bash
helm install cu-ms-payments ./helm-microservice-chart \
  --set name=cu-ms-payments \
  --set namespace=edisonpaul4-dev \
  --set registry=edisonpaul4 \
  --set image=cu-ms-payments \
  --set imageTag=v1.0.0
```

## Configuración

### Parámetros principales

| Parámetro   | Descripción                                            | Valor por defecto |
| ----------- | ------------------------------------------------------ | ----------------- |
| `name`      | Nombre del microservicio                               | `""` (requerido)  |
| `namespace` | Namespace de Kubernetes                                | `""` (requerido)  |
| `registry`  | Registry de Docker (ej: edisonpaul4, myacr.azurecr.io) | `""` (requerido)  |
| `image`     | Nombre de la imagen Docker                             | `""` (requerido)  |
| `imageTag`  | Tag de la imagen Docker                                | `""` (requerido)  |
| `port`      | Puerto del contenedor                                  | `3000`            |

### Parámetros del HPA

| Parámetro               | Descripción               | Valor por defecto |
| ----------------------- | ------------------------- | ----------------- |
| `hpa.minReplicas`       | Mínimo de réplicas        | `2`               |
| `hpa.maxReplicas`       | Máximo de réplicas        | `6`               |
| `hpa.cpuUtilization`    | % de CPU para escalar     | `90`              |
| `hpa.memoryUtilization` | % de memoria para escalar | `90`              |

### ConfigMap (variables de entorno)

| Parámetro   | Descripción                                   | Valor por defecto |
| ----------- | --------------------------------------------- | ----------------- |
| `configMap` | Mapa de llave-valor para variables de entorno | `{}`              |

### Secret (variables sensibles)

| Parámetro | Descripción                                          | Valor por defecto |
| --------- | ---------------------------------------------------- | ----------------- |
| `secret`  | Mapa de llave-valor para secrets (valores en base64) | `{}`              |

## Ejemplos de uso

### Ejemplo 1: values.yaml completo

```yaml
name: cu-ms-payments
namespace: edisonpaul4-dev
registry: edisonpaul4
image: cu-ms-payments
imageTag: v1.0.0
port: 3000

configMap:
  LOG_LEVEL: "debug"
  API_URL: "https://api.example.com"
  TIMEOUT: "30"
  NODE_ENV: "production"

secret:
  DATABASE_PASSWORD: U3VwZXJTZWd1cm8yMDI1
  API_KEY: bXlzZWNyZXRhcGlrZXk=
  JWT_SECRET: c2VjcmV0and0

hpa:
  minReplicas: 2
  maxReplicas: 10
  cpuUtilization: 80
  memoryUtilization: 85
```

### Ejemplo 2: Instalación con archivo values

```bash
helm install cu-ms-payments ./helm-microservice-chart -f values.yaml
```

### Ejemplo 3: Instalación con parámetros inline

```bash
helm install cu-ms-payments ./helm-microservice-chart \
  --set name=cu-ms-payments \
  --set namespace=edisonpaul4-dev \
  --set registry=edisonpaul4 \
  --set image=cu-ms-payments \
  --set imageTag=v1.0.0 \
  --set port=8080 \
  --set configMap.LOG_LEVEL=info \
  --set configMap.NODE_ENV=production \
  --set secret.DATABASE_PASSWORD=U3VwZXJTZWd1cm8yMDI1 \
  --set hpa.minReplicas=3 \
  --set hpa.maxReplicas=15
```

### Ejemplo 4: Actualizar un release existente

```bash
helm upgrade cu-ms-payments ./helm-microservice-chart \
  --set imageTag=v1.1.0
```

### Ejemplo 5: Instalación en namespace diferente

```bash
helm install cu-ms-orders ./helm-microservice-chart \
  --set name=cu-ms-orders \
  --set namespace=edisonpaul4-prod \
  --set registry=edisonpaul4 \
  --set image=cu-ms-orders \
  --set imageTag=v2.0.0 \
  --set configMap.LOG_LEVEL=warn \
  --set hpa.minReplicas=5 \
  --set hpa.maxReplicas=20
```

## Generar secrets en base64

Para generar valores en base64 para el Secret:

```bash
# Linux/macOS
echo -n "mi-password-seguro" | base64

# PowerShell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("mi-password-seguro"))
```

## Validar el chart antes de instalar

```bash
# Renderizar templates sin instalar
helm template cu-ms-payments ./helm-microservice-chart -f values.yaml

# Validar sintaxis
helm lint ./helm-microservice-chart

# Dry-run de instalación
helm install cu-ms-payments ./helm-microservice-chart -f values.yaml --dry-run
```

## Desinstalar

```bash
helm uninstall cu-ms-payments
```

## Estructura del Chart

```
helm-microservice-chart/
├── Chart.yaml
├── README.md
├── values.yaml
└── templates/
    ├── configmap.yaml
    ├── deployment.yaml
    ├── hpa.yaml
    ├── secret.yaml
    └── service.yaml
```

## Probes de salud

El Deployment incluye tres tipos de probes:

- **startupProbe**: `/startup` - Verifica que la aplicación inició correctamente
- **livenessProbe**: `/liveness` - Verifica que la aplicación está viva
- **readinessProbe**: `/readiness` - Verifica que la aplicación está lista para recibir tráfico

Asegúrate de que tu microservicio exponga estos endpoints.

## Recursos del contenedor

Por defecto, el contenedor tiene configurados:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "300m"
    memory: "512Mi"
```

## Licencia

MIT
