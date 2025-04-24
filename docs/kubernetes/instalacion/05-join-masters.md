---
sidebar_position: 5
title: Unir Nodos Control Plane Adicionales
---

# Unir Nodos Control Plane Adicionales (k8s-master-02, k8s-master-03)

Después de inicializar el primer Control Plane (`k8s-master-01`), une los otros nodos Control Plane (`k8s-master-02` y `k8s-master-03`) al cluster. Esto formará el plano de control de Alta Disponibilidad.

**Pasos en cada nodo Control Plane ADICIONAL (192.168.100.13 y 192.168.100.14):**

1.  **Asegúrate de que los pre-requisitos estén listos** (Docker/Containerd, kubeadm, kubelet, kubectl instalados, swap deshabilitado, /etc/hosts configurado, firewall básico listo).
    [Tu descripción aquí sobre si ya hiciste los pre-requisitos en estos nodos]

2.  **Ejecuta el comando `kubeadm join` con las flags para Control Plane.**
    Usa el comando `kubeadm join` que obtuviste como salida de `kubeadm init` en el primer maestro, asegurándote de incluir las flags `--control-plane` y `--certificate-key`.

    ```bash
    # Conectado por SSH a k8s-master-02 (192.168.100.13) Y LUEGO en k8s-master-03 (192.168.100.14)
    sudo kubeadm join 192.168.100.7:6443 --token abcdef.1234567890abcdef \
        --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
        --control-plane --certificate-key yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
    ```
    * `192.168.100.7:6443`: La IP y puerto de tu balanceador de carga.
    * `--token ...`: El token de descubrimiento (válido por 24h).
    * `--discovery-token-ca-cert-hash ...`: El hash del certificado CA del cluster.
    * `--control-plane`: Indica que este nodo se unirá como Control Plane.
    * `--certificate-key ...`: La clave para desencriptar y obtener los certificados del plano de control subidos.

    [Tu salida completa del comando sudo kubeadm join en k8s-master-02 aquí]
    [Tu salida completa del comando sudo kubeadm join en k8s-master-03 aquí]

3.  **Configura `kubectl` en los nuevos Control Planes (opcional).**
    Si quieres poder ejecutar `kubectl` directamente desde estos nuevos maestros (no es estrictamente necesario si usas kubectl desde el primero o desde local), repite los pasos de configuración de kubectl que hiciste para el primer maestro.

    ```bash
    # En k8s-master-02 Y k8s-master-03
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown <span class="math-inline">\(id \-u\)\:</span>(id -g) $HOME/.kube/config
    ```

4.  **Verifica el estado de los nodos Control Plane.**
    Desde tu máquina local (con kubectl configurado) o desde cualquiera de los maestros, verifica que los 3 Control Planes se unieron correctamente.

    ```bash
    # Desde tu máquina local o un maestro
    kubectl get nodes
    ```
    [Tu salida de kubectl get nodes aquí]

    Los 3 nodos Control Plane deben aparecer en estado `Ready`.

---