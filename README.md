# 🚀 Innovatech Chile — Frontend (Despacho Dashboard)

## Descripción General

Este repositorio contiene el código fuente y la configuración de contenedorización del **Frontend** del sistema de gestión de despachos para el proyecto Innovatech Chile (Grupo Cordillera). La aplicación está desarrollada en **React + Vite** y se despliega automáticamente en una instancia EC2 en AWS mediante un pipeline CI/CD con GitHub Actions y Amazon ECR.

---

## Arquitectura del Sistema

```
Internet
    │
    ▼
[EC2 Frontend – Subred Pública]
  nginx-unprivileged:alpine (puerto 80:8080)
    │  (HTTP hacia subred privada)
    ▼
[EC2 Backend – Subred Privada]
  Spring Boot API (puerto 8080)
  MySQL 8.0 (puerto 3306)
    │
[Volumen Docker: db_data]
```

### Componentes de la Infraestructura

| Componente | Tecnología | Instancia AWS |
|---|---|---|
| Frontend | React + Vite + Nginx | EC2 t2.micro (Subred Pública) |
| Backend Despachos | Spring Boot (Java 17) | EC2 t3.small (Subred Privada) |
| Backend Ventas | Spring Boot (Java 17) | EC2 t3.small (Subred Privada) |
| Base de Datos | MySQL 8.0 | Contenedor en EC2 Backend |
| Registro de Imágenes | Amazon ECR | us-east-1 |
| Pipeline CI/CD | GitHub Actions | Rama `deploy` |

---

## Contenedorización

### Dockerfile (Multi-Stage Build)

El Dockerfile del Frontend implementa un **multi-stage build** con dos etapas claramente definidas:

**Etapa 1 — Build (Construcción):** Utiliza `node:18-alpine` para compilar el proyecto con `npm run build`, generando los artefactos estáticos optimizados.

**Etapa 2 — Producción (Run):** Utiliza `nginxinc/nginx-unprivileged:alpine`, una imagen oficial preconfigurada con usuario no root, que expone el puerto 8080 en lugar del puerto 80 privilegiado. Esto cumple con el principio de mínimo privilegio exigido en la rúbrica.

**Beneficios del diseño:**
- Imagen final liviana (solo Nginx + archivos estáticos, sin Node.js)
- Capas limpias: el artefacto de build se copia y las herramientas de compilación se descartan
- Sin ejecución como root → menor superficie de ataque

---

## Cómo Ejecutar Localmente

### Prerrequisitos

- Docker Desktop instalado
- Docker Compose instalado
- Git

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/1Fnx/innovatech-frontend.git
cd innovatech-frontend

# 2. Configurar variables de entorno (crear .env en la raíz)
# VITE_API_URL=http://<IP_BACKEND>:8080

# 3. Construir y levantar el contenedor
docker compose up --build

# 4. Acceder desde el navegador
# http://localhost:80
```

---

## Pipeline CI/CD (GitHub Actions)

El pipeline se activa automáticamente al realizar un `push` sobre la rama **`deploy`**.

### Flujo Completo: Build → Push → Deploy

```
Push en rama deploy
        │
        ▼
[1] Checkout del código
        │
        ▼
[2] Configurar credenciales AWS (GitHub Secrets)
        │
        ▼
[3] Login a Amazon ECR
        │
        ▼
[4] docker build + docker push → ECR (front-despacho)
        │
        ▼
[5] SSH a EC2 Frontend
        │
        ▼
[6] docker compose pull + docker compose up -d
        │
        ▼
✅ Frontend actualizado en producción
```

### GitHub Secrets Requeridos

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Clave de acceso AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Clave secreta AWS |
| `AWS_SESSION_TOKEN` | Token de sesión (Lab) |
| `EC2_HOST` | IP pública de la instancia Frontend |
| `EC2_SSH_KEY` | Llave privada SSH para acceso a EC2 |

---

## Estructura del Repositorio

```
innovatech-frontend/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Pipeline GitHub Actions
├── public/
├── src/
│   └── ...                     # Código fuente React
├── Dockerfile                  # Multi-stage build
├── docker-compose.yml          # Orquestación local
├── package.json
├── vite.config.js
└── README.md
```

---

## Principios DevOps Aplicados

- **Contenedorización:** Multi-stage Dockerfile con usuario no root
- **Automatización CI/CD:** GitHub Actions activa build/push/deploy en cada push a `deploy`
- **Gestión de Secretos:** Credenciales AWS almacenadas como GitHub Secrets (nunca en código)
- **Registro de Imágenes:** Amazon ECR como registro privado y seguro
- **Control de Versiones:** Commits descriptivos con mensajes `feat:`, `fix:`, `chore:`

---

## Equipo de Desarrollo

**Grupo Cordillera — Innovatech Chile**  
Asignatura: ISY1101 — Introducción a Herramientas DevOps  
Evaluación Parcial N°2 | 2026
