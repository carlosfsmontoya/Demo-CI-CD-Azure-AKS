# Pipeline de CI/CD hacia Azure Kubernetes Service (AKS) con GitHub Actions

Este documento explica el proceso de Integración Continua y Despliegue Continuo (CI/CD) definido en el flujo de trabajo de GitHub Actions para desplegar una aplicación web en Azure Kubernetes Service (AKS). La aplicación se construye utilizando Astro, un moderno framework de front-end, y se empaqueta en un contenedor Docker. A continuación, encontrarás pasos detallados sobre cómo replicar este proceso, incluyendo la configuración de los requisitos previos necesarios, la comprensión del archivo de flujo de trabajo y la obtención de los secretos requeridos.

## Requisitos Previos

- **Cuenta de Azure**: Para desplegar la aplicación en AKS y almacenar la imagen Docker en Azure Container Registry (ACR).
- **Docker Instalado**: Para construir y empujar la imagen Docker.
- **Astro Instalado**: Asegúrate de tener Astro instalado ejecutando `npm install astro` en el directorio de tu proyecto.

## Visión General del Flujo de Trabajo

El archivo `.github/workflows/ci-cd-azure.yml` define el pipeline de CI/CD con dos trabajos principales: `continuous-integration` y `continuous-deployment`.

### Integración Continua

1. **Checkout del Código**: Clona el repositorio en el ejecutor de GitHub Actions.
2. **Inicio de Sesión en Azure Container Registry (ACR)**: Utiliza la acción `azure/docker-login` para autenticarse en ACR usando secretos.
3. **Construir y Empujar la Imagen Docker**: Construye una imagen Docker a partir del Dockerfile, la etiqueta y la empuja a ACR.

### Despliegue Continuo

1. **Checkout del Código**: Clona el repositorio en el ejecutor de GitHub Actions.
2. **Inicio de Sesión en Azure**: Inicia sesión en Azure utilizando la acción `azure/login` con credenciales almacenadas como secretos.
3. **Configuración de Kubectl**: Instala kubectl utilizando la acción `azure/setup-kubectl`.
4. **Conexión a AKS**: Utiliza la acción `azure/aks-set-context` para establecer el contexto al clúster de AKS especificado.
5. **Despliegue en AKS**: Aplica la configuración de despliegue de Kubernetes usando `kubectl apply`.
6. **Reinicio del Clúster de Kubernetes**: Reinicia el despliegue para asegurar que se use la última imagen.
7. **Obtener la IP Pública del Servicio**: Recupera la dirección IP pública del servicio desplegado y la muestra para su verificación.

## Configuración de Secretos

El flujo de trabajo requiere varios secretos que deben configurarse en tu repositorio de GitHub:

- `ACR_USERNAME` y `ACR_PASSWORD`: Credenciales para iniciar sesión en Azure Container Registry.
- `AZURE_CREDENTIALS`: Un objeto JSON que contiene tu principal de servicio de Azure para iniciar sesión en Azure.
- `AZURE_SUBSCRIPTION_ID`: Tu ID de suscripción de Azure.

Puedes crear estos secretos navegando a Configuración > Secretos en tu repositorio y haciendo clic en "Nuevo secreto de repositorio".

Para obtener y configurar los secretos necesarios (`ACR_USERNAME`, `ACR_PASSWORD`, `AZURE_CREDENTIALS`, `AZURE_SUBSCRIPTION_ID`) en GitHub, sigue estos pasos:

### Obtener `ACR_USERNAME` y `ACR_PASSWORD`

1. **Iniciar sesión en Azure CLI**:
   ```bash az login```
Para obtener y configurar los secretos necesarios (`ACR_USERNAME`, `ACR_PASSWORD`, `AZURE_CREDENTIALS`, `AZURE_SUBSCRIPTION_ID`) en GitHub, sigue estos pasos:

### Obtener `ACR_USERNAME` y `ACR_PASSWORD`

1. **Iniciar sesión en Azure CLI**:
   ```bash
   az login
   ```
2. **Obtener credenciales de ACR**:
   ```bash
   az acr credential show --name <NombreDeTuACR>
   ```
   Reemplaza `<NombreDeTuACR>` con el nombre de tu Azure Container Registry. Este comando te proporcionará el `username` (ACR_USERNAME) y `passwords` (ACR_PASSWORD).

### Obtener `AZURE_CREDENTIALS`

1. **Crear un principal de servicio**:
   ```bash
   az ad sp create-for-rbac --name "<tuApp>" --sdk-auth
   ```
   Este comando generará un objeto JSON. Copia este objeto completo; será tu valor de `AZURE_CREDENTIALS`.

### Obtener `AZURE_SUBSCRIPTION_ID`

1. **Listar suscripciones**:
   ```bash
   az account list --query "[].{name:name, subscriptionId:id}"
   ```
   Este comando te mostrará todas tus suscripciones junto con sus IDs. Copia el `id` correspondiente a tu suscripción; este será tu `AZURE_SUBSCRIPTION_ID`.

### Configuración de Secretos en GitHub

Una vez obtenidos los valores:

1. Navega a tu repositorio en GitHub.
2. Ve a "Settings" > "Secrets" > "Actions".
3. Haz clic en "New repository secret".
4. Añade cada uno de los valores obtenidos (`ACR_USERNAME`, `ACR_PASSWORD`, `AZURE_CREDENTIALS`, `AZURE_SUBSCRIPTION_ID`) como un secreto separado, proporcionando un nombre (clave) y su valor correspondiente.

## Ejecutando el Flujo de Trabajo

Para activar el flujo de trabajo, realiza un commit en la rama `main`. El pipeline de CI/CD se iniciará automáticamente, construyendo tu imagen Docker, empujándola a ACR y desplegándola en AKS.

## Conclusión

Esta guía proporciona una visión completa de cómo configurar un pipeline de CI/CD usando GitHub Actions para desplegar una aplicación en Azure Kubernetes Service. Siguiendo estos pasos, puedes automatizar el proceso de despliegue, asegurando que tu aplicación esté siempre actualizada con los últimos cambios en tu repositorio.