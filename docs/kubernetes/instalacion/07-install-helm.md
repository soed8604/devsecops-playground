---
sidebar_position: 7
title: Instalar Helm
---

# Instalar Helm

Helm es el gestor de paquetes para Kubernetes, lo que facilita la instalación y gestión de aplicaciones y servicios preconfigurados (Charts).

Instalar Helm es necesario para poder desplegar aplicaciones como el Dashboard de Kubernetes de forma sencilla.

**Pasos en tu máquina local (donde ejecutas kubectl):**

[Tu descripción aquí sobre por qué elegiste este método (ej: APT, script, binario descargado)]

**Método Recomendado (Usando APT en Ubuntu/Debian):**

1.  Añade la clave GPG del repositorio de Helm:
    ```bash
    curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
    ```
2.  Añade el repositorio de Helm:
    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
    ```
3.  Actualiza tu lista de paquetes:
    ```bash
    sudo apt update
    ```
4.  Instala Helm:
    ```bash
    sudo apt install -y helm
    ```

**Método Alternativo (Usando Script get-helm-3):**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Verificar la instalación:
```bash
helm version
```
Una vez que Helm está instalado, puedes empezar a añadir repositorios de charts y desplegar aplicaciones.