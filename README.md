# 🛒 TechRetail – Docker Swarm Deployment

Proyecto de orquestación de contenedores con Docker Swarm para la empresa de comercio electrónico TechRetail. Implementado como actividad práctica de la asignatura de Infraestructura de Tecnologías de la Información.

---

## 📋 Descripción

TechRetail es una empresa peruana de e-commerce que migró su infraestructura a una arquitectura de microservicios contenerizada usando **Docker Swarm**, logrando alta disponibilidad, escalado horizontal y balanceo de carga automático.

---

## 🏗️ Arquitectura del Clúster

```
┌─────────────────────────────────────────────────────┐
│              DOCKER SWARM CLUSTER                   │
│                                                     │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐      │
│  │  Manager  │   │ Worker 1  │   │ Worker 2  │      │
│  │   Node    │   │   Node    │   │   Node    │      │
│  └─────┬─────┘   └─────┬─────┘   └─────┬─────┘      │
│        │               │               │            │
│  ┌─────▼───────────────▼───────────────▼──────┐     │
│  │       Red Overlay (techretail_net)         │     │
│  └──────┬──────────┬──────────┬──────────┬────┘     │
│         │          │          │          │          │
│   [frontend]  [backend]  [database]  [redis]        │
│   3 réplicas  2 réplicas  1 réplica   1 réplica     │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Servicios Desplegados

| Servicio     | Imagen                        | Réplicas | Puerto |
|--------------|-------------------------------|----------|--------|
| Frontend     | nginx:alpine                  | 3        | 80     |
| Backend      | node:18-alpine                | 2        | 3000   |
| Database     | mysql:8                       | 1        | 3306   |
| Cache        | redis:7-alpine                | 1        | 6379   |
| Visualizer   | dockersamples/visualizer      | 1        | 8080   |

---

## ⚙️ Requisitos Previos

- Docker Engine instalado
- Acceso a [iximiuz Labs](https://labs.iximiuz.com/playgrounds/docker-swarm) o 3 máquinas/VMs con Docker

---

## 📦 Instrucciones de Despliegue

### 1. Inicializar el clúster Swarm (en el nodo Manager)
```bash
docker swarm init --advertise-addr <IP_MANAGER>
```

### 2. Unir los Workers al clúster
```bash
# Obtener el token
docker swarm join-token worker

# En cada nodo worker:
docker swarm join --token <TOKEN> <IP_MANAGER>:2377
```

### 3. Verificar los nodos
```bash
docker node ls
```

### 4. Crear el Secret de base de datos
```bash
echo "MiPasswordSegura123" | docker secret create db_password -
```

### 5. Desplegar el Stack
```bash
docker stack deploy -c docker-compose.yml techretail
```

### 6. Verificar los servicios
```bash
docker stack services techretail
```

### 7. Escalar el Frontend dinámicamente
```bash
docker service scale techretail_frontend=5
```

### 8. Ver logs de un servicio
```bash
docker service logs techretail_backend
```

---

## 📊 Comandos de Monitoreo

```bash
# Ver todos los nodos
docker node ls

# Ver servicios del stack
docker stack services techretail

# Ver réplicas del frontend
docker service ps techretail_frontend

# Ver réplicas del backend
docker service ps techretail_backend

# Ver secrets creados
docker secret ls
```

---

## 🧹 Limpieza

```bash
# Eliminar el stack
docker stack rm techretail

# Salir del clúster (en cada nodo)
docker swarm leave --force
```

---

## 📁 Estructura del Repositorio

```
techretail-docker-swarm/
│
├── docker-compose.yml    # Configuración del stack completo
└── README.md             # Este archivo
```

---

## 🔐 Seguridad

- Las credenciales de base de datos se gestionan con **Docker Secrets** (nunca en texto plano).
- La contraseña no está hardcodeada en el `docker-compose.yml`.
- El secret `db_password` debe crearse manualmente antes del despliegue.

---

## 👥 Autor

Proyecto desarrollado como parte de la actividad práctica de Docker Swarm.
