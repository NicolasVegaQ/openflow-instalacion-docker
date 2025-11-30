# 🚀 Instalación Completa de OpenFlow + MongoDB ReplicaSet + RabbitMQ + Traefik usando EasyPanel y Docker Swarm

Este documento describe paso a paso cómo desplegar la plataforma **OpenFlow (OpenRPA Server)** utilizando:

- Docker Swarm  
- Traefik v3  
- EasyPanel  
- MongoDB con ReplicaSet (rs0)  
- RabbitMQ con vhost  
- Dominios personalizados con SSL (ACME)

Esta configuración es **estable**, **segura**, **100% compatible con OpenRPA** y funciona en cualquier VPS Linux.

---

# 📦 Arquitectura General

```
Cliente → Traefik HTTPS → OpenFlow → MongoDB rs0 + RabbitMQ
                            ↑
                        OIDC / Token
```

Servicios usados:

| Servicio      | Descripción                                  |
|---------------|----------------------------------------------|
| **Traefik**   | Reverse proxy y certificados SSL             |
| **OpenFlow**  | API principal del ecosistema OpenRPA         |
| **MongoDB rs0** | Base de datos con ReplicaSet requerido por OpenFlow |
| **RabbitMQ**  | Sistema de colas y mensajería                |
| **EasyPanel** | Orquestación gráfica sobre Docker Swarm      |

---

# 🌐 Dominios requeridos

Configurar en tu proveedor DNS (Cloudflare, Hostinger, etc):

```
panelrpaopen.midominio.com   → para OpenFlow
paneln8n.midominio.com       → para N8N (opcional)
mq.midominio.com             → para RabbitMQ (opcional)
```

Ejemplo real usado:

```
panelrpaopen.midominio.com 
paneln8n.midominio.com 
mq.midominio.com 
```

Todos apuntan a la IP pública:

```
A   nombre   173.212.248.141
```

---

# 🔐 Certificados SSL (Traefik)

EasyPanel configura automáticamente SSL con Let's Encrypt.  
Solo asegúrate que:

1. El dominio está agregado al servicio correcto.
2. HTTPS está activado.
3. El certificado está en estado **válido**.

---

# 🗄 Configuración de MongoDB (ReplicaSet)

Crear un servicio en EasyPanel:

**Nombre:** mongodb  
**Imagen:** `mongo`  
**Comando de inicio:**

```
--bind_ip_all --replSet rs0
```

**Variables de entorno:**

```
MONGO_REPLICA_SET_NAME=rs0
```

**Verificar ReplicaSet:**

```bash
docker exec -it <ID_CONTAINER> mongosh --eval "rs.status()"
```

Debe mostrar:

```
stateStr: "PRIMARY"
```

---

# 📨 Configuración RabbitMQ

En EasyPanel → Servicio RabbitMQ:

Variables:

```
RABBITMQ_DEFAULT_USER=admin
RABBITMQ_DEFAULT_PASS=admin123
RABBITMQ_DEFAULT_VHOST=myvhost
```

URL de conexión para OpenFlow:

```
amqp_url=amqp://admin:admin123@rabbitmq:5672/myvhost
```

---

# ⚙️ Configuración completa del servicio OpenFlow

En EasyPanel → Servicio → Variables → pegar:

```env
protocol=https
port=3000
domain=panelrpaopen.midominio.com 

auto_create_users=true
auto_create_domains=true
log_with_colors=true

mongodb_url=mongodb://mongodb:27017/?authSource=admin&replicaSet=rs0
mongodb_db=openflow

amqp_url=amqp://admin:admin123@rabbitmq:5672/myvhost

agent_oidc_issuer=https://panelrpaopen.comuauto.site/oidc
agent_oidc_authorization_endpoint=https://panelrpaopen.midominio.com /oidc/auth
agent_oidc_userinfo_endpoint=http://localhost:3000/oidc/me
agent_oidc_token_endpoint=http://localhost:3000/oidc/token

agent_docker_entrypoints=web,websecure
agent_docker_certresolver=myresolver

websocket_package_size=25000
websocket_max_package_count=1048576

aes_secret=O1itlrmA47WzxPj95YHD2sZs7IchYaQI25mQ
```

⚠️ **IMPORTANTE:**  
Sin `aes_secret` o `auto_create_domains=true`, OpenFlow falla con:

```
TypeError: Cannot read properties of undefined (reading 'substring')
```

---

# ▶ Reiniciar servicios

Cuando actualices variables:

```bash
docker service update --force openrpa_ia_openflow
```

o desde EasyPanel → Reiniciar servicio.

---

# 🧪 Verificar que OpenFlow arrancó correctamente

Logs:

```bash
docker service logs openrpa_ia_openflow -f
```

Debe mostrar:

```
Connected to mongodb
Connected to rabbitmq
Root token created
OpenFlow listening on port 3000
```

---

# 🌍 Acceso al panel de OpenFlow

Abrir:

```
https://panelrpaopen.midominio.com
```

Si ves la interfaz → instalación completada.

---

# 🤖 Conectar Agentes OpenRPA

En OpenRPA:

1. Configuración
2. Server URL:
   ```
   https://panelrpaopen.midominio.com
   ```
3. Login con el usuario creado automáticamente.

---

# 📚 Créditos y agradecimientos

Documentación elaborada durante la instalación real en un entorno EasyPanel, integrando:

- Docker Swarm  
- Traefik v3  
- MongoDB ReplicaSet  
- RabbitMQ + Vhost  
- OpenFlow / OpenRPA  

Optimizado para producción.

---

# 💬 Soporte

Si deseas agregar:

- Backups automáticos  
- Monitorización (Prometheus + Loki + Grafana)  
- Workers distribuidos  
- Cluster HA  

---

### 🛠️ Elaborado por  
**Nicolás Vega Quevedo**  
Desarrollador RPA • DevOps Junior  

📩 Contacto y soporte: **nicolasvegaquevedo12@gmail.com**

---

