# OpenCore / OpenFlow en Dokploy

## Descripción

Esta instalación permite desplegar OpenCore/OpenFlow utilizando MongoDB y RabbitMQ sobre Dokploy.

La arquitectura utilizada es:

OpenFlow → RabbitMQ → MongoDB ReplicaSet

MongoDB debe funcionar obligatoriamente como ReplicaSet (`rs0`), incluso si solo existe un nodo.

---

# Requisitos Previos

## Servidor

Se recomienda:

- 4 vCPU mínimo
- 8 GB RAM mínimo
- Ubuntu 22.04 o superior
- Docker instalado
- Dokploy instalado y funcionando

---

## Dominio

Antes de desplegar:

1. Crear un subdominio.

Ejemplo:

```text
opencore.midominio.com
```

2. Crear el registro DNS apuntando a la IP pública del servidor.

Ejemplo:

```text
Tipo: A
Nombre: opencore
Valor: IP_DEL_SERVIDOR
```

3. Esperar propagación DNS.

---

# Configuración en Dokploy

## Tipo de aplicación

Crear una aplicación tipo:

```text
Compose
```

o

```text
Docker Raw Compose
```

---

## Dominio

Una vez desplegado el stack:

```text
Host:
opencore.midominio.com

Service:
openflow

Container Port:
3000

HTTPS:
Enabled
```

Importante:

El puerto del dominio siempre debe apuntar al puerto interno del contenedor:

```text
3000
```

aunque externamente OpenFlow esté publicado en otro puerto.

---

# Variables importantes

## domain

Debe contener exactamente el mismo dominio configurado en Dokploy.

Ejemplo:

```text
domain=opencore.midominio.com
```

---

## aes_secret

Clave utilizada internamente por OpenFlow.

Ejemplo:

```text
aes_secret=MiClaveSuperSeguraDe32CaracteresOMas
```

Importante:

No cambiar esta clave después de tener usuarios creados.

---

## MongoDB

La conexión debe utilizar ReplicaSet.

Ejemplo:

```text
mongodb://mongodb:27017/?replicaSet=rs0
```

---

## RabbitMQ

Ejemplo:

```text
amqp://admin:admin123@rabbitmq:5672/myvhost
```

---

# Verificaciones después del despliegue

## MongoDB

Ingresar al contenedor:

```bash
docker exec -it <contenedor_mongodb> mongosh
```

Ejecutar:

```javascript
rs.status()
```

Resultado esperado:

```text
PRIMARY
```

Si aparece:

```text
NoReplicationEnabled
```

MongoDB no quedó configurado correctamente.

---

## OpenFlow

Revisar logs:

```bash
docker logs <contenedor_openflow>
```

Debe aparecer algo similar a:

```text
Connected to mongodb
Connected to rabbitmq
VERSION:
```

Si ambos mensajes aparecen, la infraestructura está funcionando.

---

# Problemas encontrados durante la instalación

## Error MongoDB

```text
NoReplicationEnabled
```

### Causa

MongoDB iniciado sin ReplicaSet.

### Solución

Agregar:

```text
--replSet rs0
```

al servicio MongoDB.

---

## Error MongoDB + Auth

```text
security.keyFile is required when authorization is enabled with replica sets
```

### Causa

Se habilitó autenticación antes de terminar la configuración del ReplicaSet.

### Solución

Configurar ReplicaSet primero y posteriormente agregar autenticación.

---

## Error Dokploy

```text
Domain is attached to service which does not exist
```

### Causa

El dominio apuntaba a un nombre de servicio inexistente.

### Solución

Verificar que el dominio apunte exactamente al servicio `openflow`.

---

## Error Docker

```text
Bind for 0.0.0.0:3000 failed: port is already allocated
```

### Causa

Dokploy utiliza el puerto 3000.

### Solución

Publicar OpenFlow en otro puerto del host.

Ejemplo:

```text
3001:3000
```

---

## Error OpenFlow

```text
Cannot read properties of null (reading 'map')
```

### Causa

Base de datos OpenFlow creada parcialmente durante pruebas anteriores.

### Solución

Eliminar la base de datos `openflow` y desplegar nuevamente.

---

# Notas

- MongoDB debe ejecutarse como ReplicaSet.
- RabbitMQ debe estar disponible antes de iniciar OpenFlow.
- No cambiar `aes_secret` una vez existan usuarios.
- El volumen `/var/run/docker.sock` permite a OpenFlow administrar contenedores Docker.
- Dokploy ya incluye Traefik, por lo que no es necesario desplegar Traefik dentro del stack.
- Si el puerto 3000 está ocupado, utilizar otro puerto para acceso directo por IP.
