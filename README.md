# TechRetail – Docker Swarm Deployment

Proyecto de orquestacion de contenedores con Docker Swarm para la empresa de comercio electronico TechRetail.
Implementado como actividad practica de la asignatura de Infraestructura de Tecnologias de la Informacion.

---

## Descripcion

TechRetail es una empresa peruana de e-commerce que migrо su infraestructura a una arquitectura de
microservicios contenerizada usando Docker Swarm, logrando alta disponibilidad, escalado horizontal
y balanceo de carga automatico.

El sistema resuelve los problemas originales de la empresa:
- Caidas frecuentes del sistema en horas pico
- Tiempos de respuesta lentos (mas de 8 segundos por peticion)
- Perdidas de S/. 15,000 por hora de inactividad
- Imposibilidad de escalar ante la demanda

---

## Arquitectura del Cluster

```
+---------------------------------------------------------+
|              DOCKER SWARM CLUSTER                       |
|                                                         |
|  +-----------+   +-----------+   +-----------+          |
|  |  Manager  |   | Worker 1  |   | Worker 2  |          |
|  |   Node    |   |   Node    |   |   Node    |          |
|  +-----+-----+   +-----+-----+   +-----+-----+          |
|        |               |               |                |
|  +-----v---------------v---------------v------+         |
|  |       Red Overlay (techretail_net)          |         |
|  +------+----------+----------+----------+----+         |
|         |          |          |          |              |
|   [frontend]  [backend]  [database]  [redis]            |
|   3 replicas  2 replicas  1 replica   1 replica         |
+---------------------------------------------------------+
```

---

## Servicios Desplegados

| Servicio   | Imagen                          | Replicas | Puerto |
|------------|---------------------------------|----------|--------|
| frontend   | nginx:alpine                    | 3        | 80     |
| backend    | node:18-alpine                  | 2        | 3000   |
| database   | mysql:8                         | 1        | 3306   |
| cache      | redis:7-alpine                  | 1        | 6379   |
| visualizer | dockersamples/visualizer        | 1        | 8080   |

---

## Requisitos Previos

- Docker Engine instalado en los nodos
- Acceso a iximiuz Labs (labs.iximiuz.com/playgrounds/docker-swarm) o 3 maquinas/VMs con Docker

Nota: La plataforma Play with Docker recomendada originalmente fue discontinuada en marzo de 2026.
Se recomienda usar iximiuz Labs como alternativa gratuita que provee un cluster de 3 nodos preconfigurado.

---

## Instrucciones de Despliegue

### 1. Inicializar el cluster Swarm (en el nodo Manager)

```bash
docker swarm init --advertise-addr <IP_MANAGER>
```

### 2. Unir los Workers al cluster

```bash
# Obtener el token en el Manager
docker swarm join-token worker

# Ejecutar en cada nodo worker
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

### 7. Escalar el Frontend dinamicamente

```bash
docker service scale techretail_frontend=5
```

### 8. Ver logs de un servicio

```bash
docker service logs techretail_backend
```

---

## Comandos de Monitoreo

```bash
# Ver todos los nodos
docker node ls

# Ver servicios del stack
docker stack services techretail

# Ver replicas del frontend
docker service ps techretail_frontend

# Ver replicas del backend
docker service ps techretail_backend

# Ver secrets creados
docker secret ls
```

---

## Limpieza

```bash
# Eliminar el stack
docker stack rm techretail

# Salir del cluster (ejecutar en cada nodo)
docker swarm leave --force
```

---

## Estructura del Repositorio

```
techretail-docker-swarm/
|
+-- docker-compose.yml    # Configuracion del stack completo
+-- README.md             # Este archivo
```

---

## Seguridad

- Las credenciales de base de datos se gestionan con Docker Secrets (nunca en texto plano).
- La contrasena no esta hardcodeada en el docker-compose.yml.
- El secret db_password debe crearse manualmente antes del despliegue.
- La red overlay techretail_net cifra la comunicacion entre contenedores en distintos nodos.

---

## Autores

Miguel Angel Carasas Pizarro
Imer Abel Quispe Quezada

Instituto de Educacion Superior Tecnologico TECSUP
Departamento de Tecnologia Digital – Diseno y Desarrollo de Software
Profesor: Jaime Farfan Madariaga
Lima, Peru – 2026
