---
sidebar_position: 1
title: Pre-requisitos del Sistema
---

# Pre-requisitos

Antes de iniciar la instalación de Kubernetes, asegúrate de que cada uno de tus servidores (tanto Control Planes como Workers) cumpla con los siguientes requisitos:

## 1. Servidores Utilizados

Describe brevemente cuántos servidores usaste, qué roles (Control Plane, Worker) tienen y sus IPs (puedes poner una tabla simple).

| Rol           | Nombre        | Dirección IP    |
|---------------|---------------|-----------------|
| Control Plane | k8s-master-01 | 192.168.100.12  |
| Control Plane | k8s-master-02 | 192.168.100.13  |
| Control Plane | k8s-master-03 | 192.168.100.14  |
| Worker        | k8s-worker-01 | 192.168.100.15  |
| Worker        | k8s-worker-02 | 192.168.100.16  |
| Load Balancer | lb-k8s        | 192.168.100.7   |


## 2. Sistema Operativo

Especifica la distribución y versión exacta del sistema operativo instalado en **todos** los servidores.

[Tu descripción aquí. Ejemplo: Todos los servidores utilizan Ubuntu Server 22.04 LTS.]

Puedes verificar la versión con:

```bash
lsb_release -a
# O
# cat /etc/os-release
```

## 3. Recursos del Sistema

Asegúrate de que cada servidor cumpla con los requisitos mínimos recomendados por Kubernetes:

- Control Planes: Al menos 2GB de RAM y 2 CPUs. 
- Workers: Al menos 1GB de RAM y 1 CPU (se recomienda más para cargas de trabajo reales).

## 4. Configuración de Red Básica

Realiza la siguiente configuración de red en cada uno de los 5 nodos de Kubernetes (maestros y workers), NO en el balanceador por ahora:

- IPs Estáticas: Confirma que cada nodo tiene una dirección IP estática dentro de tu subred de laboratorio (192.168.100.0/24).
[Tu descripción aquí. Ejemplo: Configuramos IPs estáticas a través de /etc/netplan/ o configuraciones de red de la VM.]

- Resolución de Nombres (/etc/hosts): Edita el archivo /etc/etc/hosts en CADA NODO de Kubernetes para que puedan resolver los nombres de host de los otros nodos de Kubernetes y la IP del balanceador.

    Añade líneas como estas al archivo /etc/etc/hosts en cada uno de los 5 nodos de Kubernetes:

    ```bash
    # Hosts del Cluster Kubernetes
    192.168.100.7   lb-k8s
    192.168.100.12  k8s-master-01
    192.168.100.13  k8s-master-02
    192.168.100.14  k8s-master-03
    192.168.100.15  k8s-worker-01
    192.168.100.16  k8s-worker-02
    ```
    Puedes verificar el contenido con:

    ```bash
    cat /etc/etc/hosts
    ```
- Puertos de Firewall (UFW): Configura el firewall básico en CADA NODO de Kubernetes. Asegúrate de que SSH esté permitido para poder acceder. Importante: Por   ahora, solo asegúrate de que tu política por defecto para el tráfico entrante sea deny y permitas SSH. Abriremos otros puertos más adelante según sea necesario para Kubernetes y Calico.

    Verifica el estado de UFW:
    ```bash
    sudo ufw status verbose
    ```
    Si necesitas permitir SSH:
     ```bash
    sudo ufw allow 22/tcp
    sudo ufw enable # Si no está activo
    sudo ufw status verbose
    ```
## 5. Deshabilitar Swap

Kubernetes requiere que la memoria swap esté deshabilitada en todos los nodos de Kubernetes (maestros y workers) para un funcionamiento correcto y predecible.

Ejecuta estos comandos en cada uno de los 5 nodos de Kubernetes:

```bash
sudo swapoff -a
```
Para que el cambio sea permanente después de reiniciar, edita el archivo /etc/fstab y comenta la línea que se refiere a la swap.
```bash
# Edita el archivo fstab (puedes usar nano, vim, etc.)
sudo nano /etc/fstab
```
Busca la línea que contiene la palabra swap y pon un # al principio para comentarla.
```bash
# /swapfile swap swap defaults 0 0   <-- Línea comentada
```
Verifica que la swap esté deshabilitada:
```bash
free -h
```
## 6. Otros Requisitos Comunes
Asegurarse de tener acceso con un usuario que pueda usar sudo en cada servidor.
Asegurarse de que la hora esté sincronizada entre los servidores (generalmente con NTP, que Ubuntu configura por defecto).