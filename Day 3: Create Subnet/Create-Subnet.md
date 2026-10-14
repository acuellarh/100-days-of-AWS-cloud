# Crear una Subnet en AWS (KodeKloud)

**Tarea:** crear la subnet `datacenter-subnet` en la **default VPC**, solo en la región `us-east-1`.

## ¿Qué es una Subnet?

- **VPC:** red privada propia dentro de AWS.
- **Subnet:** una porción de la VPC. Las instancias EC2 siempre se lanzan dentro de una subnet.

**Conceptos clave:**

- **CIDR:** rango de IPs de una red. A mayor número tras la `/`, menos IPs.

  | CIDR | IPs |
  |------|-----|
  | `/16` | 65.536 |
  | `/20` | 4.096 |
  | `/24` | 256 (AWS reserva 5) |

- La subnet debe **caber dentro de la VPC** y **no solaparse** con otras.
- **Zona de disponibilidad (AZ):** cada subnet vive en una sola AZ (`us-east-1a`, `us-east-1b`...).
- **Default VPC:** rango `172.31.0.0/16`, con una subnet `/20` por AZ ya creada (`172.31.0.0/20` a `172.31.80.0/20`). Por eso se usa un rango libre: `172.31.96.0/24`.
- **Nombre = tag `Name`:** la consola lo crea sola; en CLI se indica con `--tag-specifications`.

## Opción 1: Consola web

1. Confirma la región **N. Virginia (us-east-1)**.
2. **VPC** → **Subnets** → **Create subnet**.
3. VPC ID: la marcada como **(default)**.
4. Subnet name: `datacenter-subnet` · AZ: sin preferencia · IPv4 CIDR: `172.31.96.0/24`
5. **Create subnet**.

> Si el CIDR se solapa, elige otro rango libre (ej. `172.31.100.0/24`).

## Opción 2: AWS CLI

```bash
# VPC por defecto en us-east-1
VPC_ID=$(aws ec2 describe-vpcs --region us-east-1 \
  --filters Name=isDefault,Values=true \
  --query 'Vpcs[0].VpcId' --output text)

# (Opcional) Ver rangos ya usados
aws ec2 describe-subnets --region us-east-1 \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[].CidrBlock' --output table

# Crear la subnet con su nombre
aws ec2 create-subnet --region us-east-1 \
  --vpc-id $VPC_ID \
  --cidr-block 172.31.96.0/24 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=datacenter-subnet}]'
```

## Verificación

```bash
aws ec2 describe-subnets --region us-east-1 \
  --filters Name=tag:Name,Values=datacenter-subnet \
  --query 'Subnets[0].[SubnetId,VpcId,CidrBlock,AvailabilityZone]' --output table
```

Debe mostrar la subnet en la default VPC con el CIDR `172.31.96.0/24`.

## Resultado

✅ Reto completado con éxito.
