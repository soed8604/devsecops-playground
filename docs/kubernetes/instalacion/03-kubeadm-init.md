---
sidebar_position: 3
title: Inicializar el Primer Control Plane
---

# Inicializar el Primer Control Plane (k8s-master-01)

Este es el paso donde inicias el cluster de Kubernetes en el primer nodo Control Plane (`k8s-master-01` con IP 192.168.100.12).

**Pasos en el primer nodo Control Plane (192.168.100.12):**

1.  **Instalar los paquetes necesarios (Docker/Containerd, kubeadm, kubelet, kubectl).**
    Asegúrate de que el entorno de ejecución de contenedores (como Docker o Containerd) y las herramientas de Kubernetes (`kubeadm`, `kubelet`, `kubectl`) estén instalados y configurados según los pasos estándar de `kubeadm` para tu distribución (Ubuntu). Esto generalmente implica añadir los repositorios de Kubernetes y Docker/Containerd y luego instalar los paquetes.

    [Tu descripción aquí sobre cómo instalaste Docker/Containerd y las herramientas de kubeadm]

    ```bash
    # Ejemplo de comandos comunes (pueden variar según tu versión/distro)
    # Instalar Containerd:
    # sudo apt update && sudo apt install -y containerd.io
    # sudo systemctl enable --now containerd

    # Añadir repos de Kubernetes y descargar paquetes:
    # sudo apt update
    # sudo apt install -y apt-transport-https ca-certificates curl
    # curl -fsSL https://pkgs.k8s.io/core:/stable:/vX.Y/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    # echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/vX.Y/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    # sudo apt update
    # sudo apt install -y kubelet kubeadm kubectl
    # sudo apt-mark hold kubelet kubeadm kubectl

    # Configurar cgroup driver para kubelet si usas containerd
    # Verificar el driver de containerd: sudo containerd config default | grep SystemdCgroup
    # Configurar kubelet para usar systemd: Editar /etc/default/kubelet o un drop-in en /etc/systemd/system/kubelet.service.d/
    ```

    [Tu salida de comandos de verificación de instalación aquí, ej: docker --version, containerd --version, kubeadm version]


2.  **Inicializar el cluster con `kubeadm init`.**
    Este es el comando clave que inicia el plano de control en este servidor. Apunta al balanceador de carga y especifica el rango de IPs para los pods.

    ```bash
    # En el primer nodo Control Plane (192.168.100.12)
    sudo kubeadm init \
      --control-plane-endpoint 192.168.100.7:6443 \
      --upload-certs \
      --pod-network-cidr 192.168.0.0/16 \
      --apiserver-advertise-address 192.168.100.12
    ```
    * `--control-plane-endpoint 192.168.100.7:6443`: Indica a todos los nodos cómo contactar al API Server (a través del balanceador). **Reemplaza con tu IP del Balanceador.**
    * `--upload-certs`: Sube los certificados necesarios al cluster para que otros Control Planes puedan unirse de forma segura.
    * `--pod-network-cidr 192.168.0.0/16`: Define el rango de IPs para la red de Pods. **Este CIDR debe coincidir con la configuración de Calico.**
    * `--apiserver-advertise-address 192.168.100.12`: La IP de este nodo Control Plane que el API Server debe usar para anunciarse (importante en setups HA). **Reemplaza con la IP de este primer maestro.**

    [Tu salida completa del comando sudo kubeadm init aquí]

3.  **Configurar `kubectl` para el usuario `k8sadmin`.**
    La salida de `kubeadm init` te dará estos comandos. Ejecútalos en el mismo nodo Control Plane para poder usar `kubectl` como usuario normal.

    ```bash
    # En el primer nodo Control Plane (192.168.100.12)
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown <span class="math-inline">\(id \-u\)\:</span>(id -g) $HOME/.kube/config
    ```

4.  **Guarda el comando `kubeadm join` para unir otros nodos.**
    La salida de `kubeadm init` también te proporcionará el comando exacto que necesitas ejecutar en los otros nodos Control Plane y Workers para unirse al cluster. Copia este comando y guárdalo para los próximos pasos. Deberá incluir el token y el hash de descubrimiento.

    ```bash
    # Ejemplo del comando join para Workers (puede variar)
    kubeadm join 192.168.100.7:6443 --token abcdef.1234567890abcdef \
        --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    ```
    Y un comando similar pero con flags adicionales para unir otros Control Planes.

    [Tu comando kubeadm join workers aquí]
    [Tu comando kubeadm join masters aquí, que incluirá --control-plane --certificate-key]


---