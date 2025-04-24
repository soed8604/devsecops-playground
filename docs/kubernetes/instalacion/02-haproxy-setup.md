---
sidebar_position: 2
title: Configuración de HAProxy Load Balancer
---

# Configuración de HAProxy Load Balancer

Configuramos un servidor dedicado para HAProxy (`lb-k8s` con IP 192.168.100.7) para balancear la carga del tráfico dirigido al API Server de Kubernetes (puerto 6443) entre los tres nodos Control Plane. Esto proporciona Alta Disponibilidad para el API Server.

**Pasos en el servidor del balanceador (192.168.100.7):**

1.  **Instalar HAProxy:**

    ```bash
    sudo apt update
    sudo apt install -y haproxy
    ```

2.  **Configurar HAProxy:**
    Edita el archivo de configuración principal de HAProxy.

    ```bash
    sudo nano /etc/haproxy/haproxy.cfg
    ```

    Añade o modifica las siguientes secciones para configurar el frontend que escucha en la IP del balanceador y el backend que lista los nodos Control Plane:

    ```cfg
    # Configuración de HAProxy para Kubernetes API Server
    # ... (otras configuraciones por defecto de HAProxy pueden estar aquí)

    # Frontend para el API Server de Kubernetes
    frontend kubernetes-api
      bind 192.168.100.7:6443  # **Reemplaza 192.168.100.7 con la IP de tu servidor balanceador**
      mode tcp
      option tcplog
      default_backend kubernetes-api-nodes

    # Backend para los nodos Control Plane (Maestros)
    backend kubernetes-api-nodes
      mode tcp
      balance roundrobin # Estrategia de balanceo (ej: roundrobin)
      option tcp-check
      server k8s-master-01 192.168.100.12:6443 check fall 2 rise 1
      server k8s-master-02 192.168.100.13:6443 check fall 2 rise 1
      server k8s-master-03 192.168.100.14:6443 check fall 2 rise 1
      # **Reemplaza IPs y nombres de host con los de tus Control Planes**
    ```

    Guarda y sal del editor.

3.  **Habilitar HAProxy para iniciar al arranque y reiniciarlo:**

    ```bash
    sudo systemctl enable haproxy
    sudo systemctl restart haproxy
    ```

4.  **Verificar el estado de HAProxy:**

    ```bash
    sudo systemctl status haproxy
    ```
    Debería mostrar `active (running)`.

    [Tu salida de sudo systemctl status haproxy aquí]

5.  **Configurar Firewall (UFW) en el servidor del balanceador:**
    Asegúrate de que el firewall en el servidor del balanceador permite el tráfico entrante en el puerto 6443 TCP desde las IPs de tus nodos de Kubernetes y desde tu máquina local (donde ejecutarás `kubectl`).

    ```bash
    # En el servidor del balanceador (192.168.100.7)
    # Permite tráfico a 6443 desde tu subred de nodos y tu IP local
    sudo ufw allow from 192.168.100.0/24 to any port 6443 proto tcp
    sudo ufw allow from 172.16.40.21 to any port 6443 proto tcp # **Reemplaza 172.16.40.21 con la IP de tu máquina local**

    # Si tienes una regla DENY Anywhere para 6443, ELIMINALA (ver troubleshooting)
    # sudo ufw delete deny 6443

    sudo ufw reload # Recarga las reglas
    sudo ufw status verbose # Verifica las reglas
    ```
    [Tu salida de sudo ufw status verbose en el LB aquí]

Con HAProxy configurado y el firewall abierto, el tráfico dirigido a 192.168.100.7:6443 será balanceado a los Control Planes.

---