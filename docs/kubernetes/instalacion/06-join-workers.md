---
sidebar_position: 6
title: Unir Nodos Workers
---

# Unir Nodos Workers (k8s-worker-01, k8s-worker-02)

Finalmente, une los nodos Worker (`k8s-worker-01` y `k8s-worker-02`) al cluster. Estos nodos ejecutarán las cargas de trabajo (tus aplicaciones en Pods).

**Pasos en cada nodo Worker (192.168.100.15 y 192.168.100.16):**

1.  **Asegúrate de que los pre-requisitos estén listos** (Docker/Containerd, kubeadm, kubelet, kubectl instalados, swap deshabilitado, /etc/hosts configurado, firewall básico listo).
    [Tu descripción aquí sobre si ya hiciste los pre-requisitos en estos nodos]

2.  **Ejecuta el comando `kubeadm join` sin flags adicionales.**
    Usa el comando `kubeadm join` que obtuviste como salida de `kubeadm init` en el primer maestro. Para los workers, no necesitas las flags `--control-plane` ni `--certificate-key`.

    ```bash
    # Conectado por SSH a k8s-worker-01 (192.168.100.15) Y LUEGO en k8s-worker-02 (192.168.100.16)
    sudo kubeadm join 192.168.100.7:6443 --token abcdef.1234567890abcdef \
        --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    ```
    * `192.168.100.7:6443`: La IP y puerto de tu balanceador de carga.
    * `--token ...`: El token de descubrimiento.
    * `--discovery-token-ca-cert-hash ...`: El hash del certificado CA del cluster.

    [Tu salida completa del comando sudo kubeadm join en k8s-worker-01 aquí]
    [Tu salida completa del comando sudo kubeadm join en k8s-worker-02 aquí]

3.  **Verifica el estado de todos los nodos.**
    Desde tu máquina local (con kubectl configurado) o desde cualquiera de los maestros, verifica que los 5 nodos se unieron correctamente.

    ```bash
    # Desde tu máquina local o un maestro
    kubectl get nodes
    ```
    [Tu salida final de kubectl get nodes aquí, mostrando los 3 maestros y 2 workers Ready]

    Todos los 5 nodos deben aparecer en estado `Ready`. Si algún worker se queda en `NotReady` después de unos minutos, puede ser un problema con la CNI en ese nodo (ver sección de Troubleshooting).

¡En este punto, tu cluster Kubernetes HA básico está instalado y los nodos están unidos!

---