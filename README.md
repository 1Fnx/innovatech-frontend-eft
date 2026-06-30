# Innovatech Chile - Frontend (Examen Final Transversal - EFT)

Repositorio del Frontend de la aplicación **Tienda de Alimentos para Perritos**, parte del caso Innovatech Chile.

Frontend simple en HTML + JavaScript vanilla servido por Nginx, contenedorizado y desplegado en AWS ECS Fargate.

## Estructura del Proyecto

```
.
├── frontend/
│   ├── Dockerfile          # Multietapa (builder + nginx runtime)
│   ├── index.html          # Estructura HTML
│   ├── app.js              # Lógica CRUD del cliente
│   └── default.conf        # Configuración Nginx
├── docker-compose.yml      # Entorno local
├── .github/workflows/
│   └── cicd-tienda-frontend.yml
└── README.md
```

## Stack Tecnológico

| Componente | Tecnología |
|---|---|
| Frontend | HTML5 + JavaScript ES6 |
| Servidor web | Nginx (alpine) |
| Build | Docker multietapa con envsubst |
| CI/CD | GitHub Actions |
| Registro de imágenes | Amazon ECR |
| Orquestación cloud | AWS ECS Fargate |

## Cómo correr el proyecto localmente

### Requisitos previos

- Docker Desktop instalado y corriendo
- El backend corriendo en otra terminal (ver repo del backend)

### Paso 1: Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/innovatech-frontend-eft.git
cd innovatech-frontend-eft
```

### Paso 2: Levantar el frontend apuntando al backend local

```bash
API_URL=http://localhost:3000/api docker-compose up --build
```

El frontend queda disponible en `http://localhost:8080`.

### Paso 3: Detener el frontend

```bash
docker-compose down
```

## Inyección de la URL de la API

Este frontend usa un mecanismo de **build-time injection** para no hardcodear la URL del backend:

1. En `app.js` la URL se declara como `const API_BASE = "${API_URL}/productos";`
2. En el Dockerfile, la etapa `builder` ejecuta `envsubst` que reemplaza `${API_URL}` por el valor real
3. La etapa `runtime` (Nginx) sirve el archivo ya procesado

Esto permite usar la **misma imagen base** con distintos backends (dev, staging, prod) cambiando solo el `--build-arg API_URL`.

### Ejemplo

```bash
# Apuntando a backend local
docker build --build-arg API_URL=http://localhost:3000/api -t frontend-local .

# Apuntando a backend en AWS
docker build --build-arg API_URL=http://alb-eft.us-east-1.elb.amazonaws.com:3000/api -t frontend-prod .
```

## Pipeline CI/CD

El pipeline (`cicd-tienda-frontend.yml`) se gatilla con cada push a `main` y ejecuta:

1. **Job `validate`**:
   - Valida que index.html no esté vacío
   - Valida sintaxis de app.js con Node
   - Valida configuración de Nginx con `nginx -t`
2. **Job `build-push-deploy`** (solo si validate pasa):
   - Login en Amazon ECR
   - Build de la imagen con `--build-arg API_URL`
   - Escaneo de vulnerabilidades con Trivy
   - Push de la imagen a ECR (tag SHA + tag latest)
   - Actualización de la Task Definition de ECS
   - Deploy con `wait-for-service-stability`

### Secrets requeridos en GitHub

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS |
| `AWS_SESSION_TOKEN` | Token de sesión (AWS Academy) |
| `AWS_REGION` | Región AWS (ej. us-east-1) |
| `API_URL` | URL pública del backend (ej. http://alb-eft...:3000/api) |

## Funcionalidades

La aplicación permite gestionar el catálogo de productos de la tienda:

- Listar productos
- Crear un producto nuevo
- Editar un producto existente
- Eliminar un producto

Cada operación dispara un fetch HTTP al backend, que persiste los datos en MySQL.

## Arquitectura en AWS

El frontend se despliega como una tarea ECS Fargate detrás del Application Load Balancer:

- **AWS ECS Fargate** ejecuta el contenedor Nginx
- **Amazon ECR** almacena la imagen del frontend
- **Application Load Balancer** expone el frontend públicamente en el puerto 80
- **Security Groups** restringen el acceso al puerto 80 vía el ALB

El diagrama detallado de arquitectura se encuentra en el informe del proyecto.
