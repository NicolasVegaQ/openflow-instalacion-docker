# openflow-instalacion-docker
---
# Configuración de DNS para el Servidor

La configuración correcta de registros DNS es clave para que los servicios sean accesibles y funcionen adecuadamente. Este archivo explica la importancia de los registros **A** y **CNAME**.

## Registros DNS

### Registro A
- **Descripción**: Apunta un dominio a una dirección IP.
- **Ejemplo**: `openflow.yourdomain.com -> 192.168.1.1`

### Registro CNAME
- **Descripción**: Crea un alias para un dominio.
- **Ejemplo**: `*.openflow.yourdomain.com -> openflow.yourdomain.com`

## Configuración

1. Crear un registro **A** para el dominio principal:
   ```plaintext
   Tipo: A
   Nombre: openflow.yourdomain.com
   Valor: 192.168.1.1

   Tipo: CNAME
    Nombre: *.openflow.yourdomain.com
    Valor: openflow.yourdomain.com

