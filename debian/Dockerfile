# -----------------------------------------------------------------------------
# Dockerfile para Entorno de Desarrollo Cloud y DevOps (Basado en Ubuntu)
# -----------------------------------------------------------------------------
#
# Descripción:
# Esta imagen de Docker proporciona un entorno de desarrollo autocontenido con
# herramientas esenciales para automatización, IaC y gestión de la nube.
#
# Versión: 1.1.0
# Base: Ubuntu 22.04 LTS

# 1. Etapa de Definición
# ---------------------
# Usamos una imagen base oficial de Ubuntu LTS (Long-Term Support)
FROM debian:12-slim

# Argumentos para definir las versiones de las herramientas, permitiendo
# personalización durante la construcción de la imagen.
ARG TERRAFORM_VERSION="1.9.0"
ARG TASK_VERSION="3.37.0"

# 2. Configuración del Entorno
# ---------------------------
# Variables de entorno para una construcción no interactiva y para asegurar
# que los ejecutables de pipx estén en el PATH del sistema.
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=America/Santiago
ENV PATH="/root/.local/bin:${PATH}"

# 3. Instalación de Dependencias del Sistema
# ------------------------------------------
# Actualizamos los repositorios e instalamos las dependencias básicas:
# - software-properties-common: Para gestionar repositorios (necesario para Ansible en algunas configuraciones)
# - curl, unzip: Para descargar y descomprimir archivos.
# - git: Sistema de control de versiones.
# - python3, python3-pip: Para Ansible y pipx.
# - nano: Editor de texto simple para ediciones rápidas.
# - jq: Procesador de JSON para la línea de comandos.
# - build-base (en Alpine) o build-essential (en Debian/Ubuntu): A veces necesario para pip.
RUN apt-get update && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    apt-get install -y \
    software-properties-common \
    curl \
    unzip \
    git \
    python3 \
    python3-pip \
    nano \
    jq \
    pipx && \
    # Limpiamos la caché de apt para reducir el tamaño de la imagen final.
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 4. Instalación de Herramientas de Aplicación
# --------------------------------------------

# Instala Ansible usando pipx para un entorno aislado y seguro.
# Esto evita conflictos de dependencias entre herramientas de Python.
RUN pipx install --pip-args='--no-cache-dir' ansible-core

# Instala AWS CLI v2 (Interfaz de Línea de Comandos de AWS).
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf aws awscliv2.zip

# Instala Terraform (Herramienta de Infraestructura como Código).
RUN curl -sL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" -o terraform.zip && \
    unzip terraform.zip -d /usr/local/bin/ && \
    rm terraform.zip

# Instala Task (Ejecutor de tareas moderno).
RUN curl -sL "https://github.com/go-task/task/releases/download/v${TASK_VERSION}/task_linux_amd64.tar.gz" -o task.tar.gz && \
    tar -xzf task.tar.gz -C /usr/local/bin/ task && \
    rm task.tar.gz

# Instala kubectl (Herramienta de línea de comandos para Kubernetes).
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# 5. Configuración Final
# ---------------------
# Define un directorio de trabajo dentro del contenedor para mantener
# los archivos del proyecto organizados.
WORKDIR /workspace

# Comando por defecto que se ejecuta al iniciar el contenedor.
# Inicia una sesión de bash para que el usuario pueda empezar a trabajar.
CMD ["/bin/bash"]