# Crear un Security Group en AWS (KodeKloud)

**Tarea:** crear el security group `xfusion-sg` con:
- Descripción: `Security group for Nautilus App Servers`
- Regla de entrada **HTTP** (puerto `80`) desde `0.0.0.0/0`
- Regla de entrada **SSH** (puerto `22`) desde `0.0.0.0/0`

## ¿Qué es un Security Group?

Un **firewall virtual** que controla el tráfico que entra y sale de las instancias EC2.

- **Inbound (entrada):** bloqueado por defecto; solo entra lo permitido.
- **Outbound (salida):** permitido por defecto.
- **Stateful:** si una petición entra, su respuesta sale automáticamente.
- **Solo permiten:** no hay reglas de "denegar"; lo no permitido queda bloqueado.
- **Pertenecen a una VPC** (red privada en AWS); se usa la **default VPC**.

| Parte de la regla | Significado | En este reto |
|-------------------|-------------|--------------|
| Tipo | Servicio / protocolo | HTTP, SSH |
| Puerto | "Puerta" de entrada | 80, 22 |
| Origen | Desde dónde (CIDR) | `0.0.0.0/0` = cualquier IP |

> ⚠️ En producción, SSH debe limitarse a tu IP (`x.x.x.x/32`), no a `0.0.0.0/0`.

## Opción 1: Consola web

1. Verifica la **región** (normalmente `us-east-1`).
2. **EC2** → **Network & Security** → **Security Groups** → **Create security group**.
3. Name: `xfusion-sg` · Description: `Security group for Nautilus App Servers` · VPC: default
4. **Inbound rules:**
   - `HTTP` · puerto 80 · Source: `0.0.0.0/0`
   - `SSH` · puerto 22 · Source: `0.0.0.0/0`
5. Outbound sin cambios → **Create security group**.

## Opción 2: AWS CLI

```bash
# VPC por defecto
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query 'Vpcs[0].VpcId' --output text)

# Crear el security group
SG_ID=$(aws ec2 create-security-group \
  --group-name xfusion-sg \
  --description "Security group for Nautilus App Servers" \
  --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Reglas de entrada
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
```

## Verificación

```bash
aws ec2 describe-security-groups --group-names xfusion-sg \
  --query 'SecurityGroups[0].[GroupName,Description,IpPermissions[].[FromPort,IpRanges[].CidrIp]]'
```

Debe mostrar `xfusion-sg`, la descripción exacta y los puertos 80 y 22 con `0.0.0.0/0`.

## Resultado

✅ Reto completado con éxito.
