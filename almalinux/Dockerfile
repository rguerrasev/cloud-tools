# -----------------------------------------------------------------------------
# Dockerfile para Entorno de Desarrollo Cloud y DevOps (Basado en CentOS)
# -----------------------------------------------------------------------------
#
# Descripción:
# Esta imagen de Docker proporciona un entorno de desarrollo autocontenido con
# herramientas esenciales para automatización, IaC y gestión de la nube.
#
# Versión: 1.3.0
# Base: CentOS Stream 9

# 1. Etapa de Definición
# ---------------------
# Usamos una imagen base oficial de CentOS Stream 9.
FROM almalinux:9

# Argumentos para definir las versiones de las herramientas.
ARG TERRAFORM_VERSION="1.9.0"
ARG TASK_VERSION="3.37.0"

# 2. Configuración del Entorno
# ---------------------------
# Variables de entorno para la zona horaria y para el PATH de pipx.
ENV TZ=America/Santiago
ENV PATH="/root/.local/bin:${PATH}"

# 3. Instalación de Dependencias del Sistema
# ------------------------------------------
# Actualizamos los repositorios e instalamos las dependencias con dnf.
# A diferencia de apt, dnf no suele requerir configuración de frontend.
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/almalinux*.repo && \
    sed -i 's|# baseurl=https://repo.almalinux.org|baseurl=https://repo.almalinux.org|g' /etc/yum.repos.d/almalinux*.repo && \
    dnf -y update && \
    dnf -y install \
    curl \
    unzip \
    git \
    python3 \
    python3-pip \
    nano \
    jq && \
    # Limpiamos la caché de dnf para reducir el tamaño de la imagen.
    dnf clean all

# 4. Instalación de Herramientas de Aplicación
# --------------------------------------------

# Instala Ansible usando pipx para un entorno aislado.
RUN pip3 install --no-cache-dir pipx && \
    pipx install --pip-args='--no-cache-dir' ansible-core

# Instala AWS CLI v2.
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf aws awscliv2.zip

# Instala Terraform.
RUN curl -sL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" -o terraform.zip && \
    unzip terraform.zip -d /usr/local/bin/ && \
    rm terraform.zip

# Instala Task.
RUN curl -sL "https://github.com/go-task/task/releases/download/v${TASK_VERSION}/task_linux_amd64.tar.gz" -o task.tar.gz && \
    tar -xzf task.tar.gz -C /usr/local/bin/ task && \
    rm task.tar.gz

# Instala kubectl.
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# 5. Configuración Final
# ---------------------
WORKDIR /workspace
CMD ["/bin/bash"]
