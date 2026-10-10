# AWS + Lambda Integration

Proyecto de infraestructura como código desarrollado con **Terraform** para implementar una arquitectura serverless en **Amazon Web Services (AWS)**.

La solución permite recibir información mediante **Amazon API Gateway**, procesarla utilizando **AWS Lambda**, almacenarla en **Amazon S3** y utilizar **Amazon SQS** para desacoplar el procesamiento asíncrono.

La arquitectura puede desplegarse de forma independiente en tres entornos:

- DEV
- QA
- PROD

La separación por ambientes permite realizar pruebas y validaciones de forma independiente antes de pasar los cambios a producción. De esta manera, DEV se utiliza para desarrollo, QA para pruebas y PROD para el entorno final.
---

## 1. Objetivo del proyecto

Implementar mediante Terraform una arquitectura AWS capaz de:

1. Recibir solicitudes HTTP mediante Amazon API Gateway.
2. Ejecutar una función Lambda de carga.
3. Almacenar los archivos recibidos en Amazon S3.
4. Enviar mensajes a una cola de Amazon SQS.
5. Procesar los mensajes mediante una segunda función Lambda.
6. Guardar los archivos procesados nuevamente en Amazon S3.
7. Registrar eventos y ejecuciones mediante Amazon CloudWatch.
8. Desplegar la solución de manera independiente en DEV, QA y PROD.
9. Eliminar completamente los recursos utilizando `terraform destroy`.

---

## 2. Arquitectura implementada

Flujo principal:

```text
Cliente
   |
   v
Amazon API Gateway
   |
   v
Lambda Upload
   |
   +--------------------+
   |                    |
   v                    v
Amazon S3            Amazon SQS
uploads/                 |
                         v
                  Lambda Processor
                         |
                         v
                     Amazon S3
                    processed/
```

La infraestructura también incluye:

- Amazon VPC
- Subredes públicas y privadas
- Internet Gateway
- NAT Gateways
- Route Tables
- VPC Endpoint para Amazon S3
- VPC Endpoint para Amazon SQS
- IAM Roles y Policies
- Amazon CloudWatch Logs
- CloudWatch Alarm para la Dead Letter Queue
- Amazon SQS Dead Letter Queue

---

## 3. Tecnologías utilizadas

- AWS
- Terraform
- AWS CLI
- AWS IAM Identity Center
- Python 3.12
- Git
- GitHub
- Windows CMD

---

## 4. Estructura del proyecto

```text
aws-lambda-integration/
│
├── environments/
│   ├── dev.tfvars
│   ├── qa.tfvars
│   └── prod.tfvars
│
├── lambda/
│   ├── upload/
│   │   └── index.py
│   │
│   └── processor/
│       └── index.py
│
├── terraform/
│   ├── api_gateway.tf
│   ├── cloudwatch.tf
│   ├── data.tf
│   ├── endpoints.tf
│   ├── iam.tf
│   ├── lambda.tf
│   ├── network.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── s3.tf
│   ├── sqs.tf
│   ├── variables.tf
│   └── versions.tf
│
├── .gitignore
└── README.md
```

---

## 5. Requisitos previos

Antes de desplegar la infraestructura es necesario tener instalado:

### AWS CLI

Verificar:

```bash
aws --version
```

### Terraform

Verificar:

```bash
terraform --version
```

### Git

Verificar:

```bash
git --version
```

También se necesita una cuenta de AWS con permisos suficientes para crear los recursos utilizados por el proyecto.

---

## 6. Autenticación con AWS IAM Identity Center

Este proyecto utiliza AWS IAM Identity Center mediante AWS CLI.

El perfil utilizado durante el desarrollo fue:

```text
betto-admin
```

Para iniciar sesión:

```bash
aws sso login --profile betto-admin
```

Para comprobar la identidad:

```bash
aws sts get-caller-identity --profile betto-admin
```

---

## 7. Inicialización de Terraform

Ingresar a la carpeta:

```bash
cd terraform
```

Inicializar Terraform:

```bash
terraform init
```

Formatear los archivos:

```bash
terraform fmt
```

Validar la configuración:

```bash
terraform validate
```

Resultado esperado:

```text
Success! The configuration is valid.
```

---

## 8. Entornos

La solución está preparada para los siguientes entornos:

```text
DEV
QA
PROD
```

Cada uno utiliza su propio archivo de variables.

### DEV

```text
../environments/dev.tfvars
```

### QA

```text
../environments/qa.tfvars
```

### PROD

```text
../environments/prod.tfvars
```

---

## 9. Despliegue DEV

DEV fue desplegado inicialmente utilizando el workspace `default`.

Seleccionar:

```bash
terraform workspace select default
```

Plan:

```bash
terraform plan -var-file="../environments/dev.tfvars"
```

Desplegar:

```bash
terraform apply -var-file="../environments/dev.tfvars"
```

Confirmar escribiendo:

```text
yes
```

---

## 10. Despliegue QA

Crear el workspace si todavía no existe:

```bash
terraform workspace new qa
```

O seleccionarlo:

```bash
terraform workspace select qa
```

Plan:

```bash
terraform plan -var-file="../environments/qa.tfvars"
```

Desplegar:

```bash
terraform apply -var-file="../environments/qa.tfvars"
```

Confirmar:

```text
yes
```

---

## 11. Despliegue PROD

Crear el workspace si todavía no existe:

```bash
terraform workspace new prod
```

O seleccionarlo:

```bash
terraform workspace select prod
```

Plan:

```bash
terraform plan -var-file="../environments/prod.tfvars"
```

Desplegar:

```bash
terraform apply -var-file="../environments/prod.tfvars"
```

Confirmar:

```text
yes
```

---

## 12. Outputs de Terraform

Después de un despliegue exitoso Terraform muestra valores como:

```text
api_url
bucket_name
dlq_url
processor_lambda_name
sqs_queue_url
upload_endpoint
upload_lambda_name
```

El output `upload_endpoint` contiene la URL utilizada para probar la integración.

---

## 13. Prueba del endpoint

Ejemplo desde Windows CMD:

```cmd
curl -X POST ^
  -H "Content-Type: text/plain" ^
  --data "Prueba AWS Lambda Integration" ^
  URL_DEL_ENDPOINT/upload
```

Respuesta esperada:

```json
{
  "message": "Archivo recibido correctamente",
  "key": "uploads/archivo.bin"
}
```

---

## 14. Verificación de Amazon S3

Para verificar los archivos:

```bash
aws s3 ls s3://NOMBRE_DEL_BUCKET/ --recursive --profile betto-admin
```

El resultado debe mostrar archivos en:

```text
uploads/
processed/
```

Ejemplo:

```text
uploads/archivo.bin
processed/archivo.bin
```

Esto demuestra que la integración completa funcionó correctamente.

---

## 15. Flujo de procesamiento

El flujo funcional es:

```text
POST /upload
     |
     v
API Gateway
     |
     v
Lambda Upload
     |
     +------> Amazon S3 /uploads
     |
     +------> Amazon SQS
                  |
                  v
           Lambda Processor
                  |
                  v
           Amazon S3 /processed
```

---

## 16. CloudWatch

Las funciones Lambda generan logs en Amazon CloudWatch.

Para revisar los logs de la función Upload:

```bash
aws logs tail "/aws/lambda/aws-lambda-integration-dev-upload" --since 10m --profile betto-admin
```

Para Processor:

```bash
aws logs tail "/aws/lambda/aws-lambda-integration-dev-processor" --since 10m --profile betto-admin
```

En Processor se puede observar una salida similar a:

```text
Procesado: uploads/archivo.bin -> processed/archivo.bin
```

---

## 17. Dead Letter Queue

La arquitectura utiliza una Dead Letter Queue para almacenar mensajes que no puedan procesarse correctamente después de varios intentos.

Terraform configura:

```text
maxReceiveCount = 3
```

Después de superar dicho número de intentos, SQS envía el mensaje a la DLQ.

---


## 18. Eliminación de recursos
=======
## 18. Verificación del despliegue

Después de ejecutar `terraform apply`, se debe comprobar en la consola de AWS que los recursos fueron creados correctamente.

Se recomienda verificar:

- Amazon API Gateway
- AWS Lambda Upload
- AWS Lambda Processor
- Amazon S3
- Amazon SQS
- Dead Letter Queue
- CloudWatch Logs
- Recursos de red de la VPC

También se debe guardar evidencia de los entornos DEV, QA y PROD antes de ejecutar `terraform destroy`.

---

## 19. Eliminación de recursos


Una parte obligatoria del proyecto es demostrar la eliminación de la infraestructura mediante Terraform.

### Destruir PROD

```bash
terraform workspace select prod
terraform destroy -var-file="../environments/prod.tfvars"
```

### Destruir QA

```bash
terraform workspace select qa
terraform destroy -var-file="../environments/qa.tfvars"
```

### Destruir DEV

```bash
terraform workspace select default
terraform destroy -var-file="../environments/dev.tfvars"
```

En todos los casos se debe confirmar escribiendo:

```text
yes
```

El resultado esperado es:

```text
Destroy complete!
```

---

## 19. Eliminación automática del bucket S3

El bucket utiliza versionado.

Para permitir que Terraform elimine también los objetos, versiones y delete markers durante `terraform destroy`, se configuró:

```hcl
force_destroy = true
```

Esto permite completar correctamente la destrucción del bucket S3.

---

## 20. Estado final del proyecto

Los tres ambientes fueron implementados y probados correctamente:

```text
DEV  -> desplegado, probado y destruido correctamente
QA   -> desplegado, probado y destruido correctamente
PROD -> desplegado, probado y destruido correctamente
```

Se verificó:

```text
API Gateway             OK
Lambda Upload           OK
Amazon S3 uploads       OK
Amazon SQS              OK
Lambda Processor        OK
Amazon S3 processed     OK
CloudWatch Logs         OK
IAM                     OK
VPC                     OK
Terraform DEV           OK
Terraform QA            OK
Terraform PROD          OK
Terraform Destroy       OK
```

---

## 21. Repositorio

Repositorio del proyecto:

https://github.com/Betto-lab/AWS-Lambda---Proyecto

---

## 22. Conclusión

El proyecto demuestra la implementación de una arquitectura AWS utilizando Infrastructure as Code mediante Terraform.

La solución permite desplegar múltiples ambientes independientes y automatiza la creación de servicios de red, almacenamiento, procesamiento serverless, mensajería, seguridad y monitoreo.

Las pruebas realizadas confirmaron el funcionamiento completo del flujo desde API Gateway hasta el procesamiento final de los archivos almacenados en Amazon S3.

Finalmente, se realizó la eliminación controlada de los recursos mediante `terraform destroy`, comprobando que toda la infraestructura puede ser creada y eliminada de manera reproducible mediante código.}


## Evidencias

El proyecto fue desplegado y validado en AWS para los ambientes DEV, QA y PROD.
(LO BORRAN)

INTEGRANTES:
Enriquez Cabanillas, César - 000280651
Lázaro Velásquez, Jesús - 000202981
Martino López, Marielsys - 000281361
Moran Carbonel, Jair - 000284492
Mori Galarza, Franco - 0000276998
