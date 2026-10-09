\# AWS + Lambda Integration



Proyecto de infraestructura como código con Terraform para desplegar una arquitectura AWS integrada con Lambda.



\## Entornos



\- DEV

\- QA

\- PROD



\## Componentes principales



\- Amazon API Gateway

\- AWS Lambda

\- Amazon S3

\- Amazon SQS

\- Dead Letter Queue

\- Amazon VPC

\- Subredes públicas y privadas

\- NAT Gateway

\- VPC Endpoints

\- IAM

\- Amazon CloudWatch



## Estado actual

- DEV desplegado y probado correctamente.
- QA desplegado y probado correctamente.
- PROD desplegado y probado correctamente.


\## Estructura



\- `terraform/` → infraestructura

\- `environments/` → variables DEV, QA y PROD

\- `lambda/` → código de las funciones Lambda

