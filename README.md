# Innovatech Chile - Frontend (EFT)

Repositorio del Frontend de la aplicación Tienda de Alimentos para Perritos. HTML + JavaScript servido por Nginx.

## Estructura del Proyecto

```
.
├── frontend/
│   ├── Dockerfile          # Multietapa (builder + nginx)
│   ├── index.html
│   ├── app.js
│   └── default.conf
├── docker-compose.yml
├── .github/workflows/
│   └── cicd-tienda-frontend.yml
└── README.md
```

## Cómo correr localmente

Requiere que el backend esté corriendo en otro terminal.

```bash
API_URL=http://localhost:3000/api docker-compose up --build
# Abrir http://localhost:8080
```

## Pipeline CI/CD

Se gatilla con cada push a main y ejecuta:

1. **Job validate**: valida HTML, JS y configuración Nginx
2. **Job build-push-deploy**: build con API_URL inyectada, escaneo Trivy, push a ECR, deploy a ECS

## Inyección de URL del Backend

La URL del backend se inyecta en build time usando envsubst. No está hardcoded en el código.

```bash
docker build --build-arg API_URL=http://mi-alb:3000/api -t frontend .
```

## Arquitectura en AWS

- AWS ECS Fargate con Nginx
- Amazon ECR
- Application Load Balancer (puerto 80)
- Security Groups restrictivos
