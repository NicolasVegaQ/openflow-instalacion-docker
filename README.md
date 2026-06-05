# OpenCore / OpenFlow + MongoDB ReplicaSet + RabbitMQ en Dokploy

## Descripción

Esta configuración permite desplegar OpenCore/OpenFlow utilizando:

- MongoDB 7
- MongoDB ReplicaSet (rs0)
- RabbitMQ Management
- OpenFlow/OpenCore
- Dokploy + Traefik

La configuración fue validada en Dokploy y permite acceder correctamente al panel web de OpenCore. OpenFlow requiere MongoDB configurado como ReplicaSet, incluso cuando se utiliza una única instancia. :contentReference[oaicite:0]{index=0}

---

# Arquitectura

```text
Internet
    │
    ▼
Traefik (Dokploy)
    │
    ▼
OpenFlow
    │
    ├── MongoDB ReplicaSet (rs0)
    │
    └── RabbitMQ
