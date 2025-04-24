---
sidebar_position: 4
title: Instalar CNI Calico
---

# Instalar CNI Calico

La CNI (Container Network Interface) es esencial para la comunicación entre los pods y para que la red de pods funcione. En este lab, usamos Calico.

**Paso en el primer nodo Control Plane (192.168.100.12) o tu máquina local con kubectl configurado:**

1.  **Descargar el manifiesto de instalación de Calico:**
    Puedes descargar el manifiesto oficial. La versión puede variar.

    ```bash
    # En la máquina donde tienes kubectl configurado (ej: primer maestro)
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
    # O si usas una versión específica recomendada por kubeadm
    # curl https://raw.githubusercontent.com/kubernetes/kubernetes/release-X.Y/cluster/addons/coredns/coredns.yaml.sed
    ```
    [Tu descripción aquí sobre la versión de Calico que usaste o por qué elegiste ese manifiesto]

2.  **Modificar el manifiesto si es necesario:**
    Si el CIDR de Pods que usaste en `kubeadm init` (192.168.0.0/16) no coincide con el CIDR predeterminado en el manifiesto de Calico (a menudo 192.168.0.0/16 o 10.42.0.0/16), **debes modificar el manifiesto de Calico** para que coincida. Busca la sección `CalicoNetwork` y el `IPPool` y asegúrate de que el campo `cidr` sea `192.168.0.0/16`.

    [Tu descripción aquí sobre si tuviste que modificar el manifiesto de Calico y qué líneas específicas cambiaste para que el campo 'cidr' del IPPool fuera 192.168.0.0/16]

3.  **Aplicar el manifiesto modificado de Calico:**

    ```bash
    # En la máquina donde tienes kubectl configurado
    kubectl apply -f calico.yaml
    ```
    [Tu salida de comando kubectl apply -f calico.yaml aquí]

4.  **Verificar el estado de los pods de Calico y CoreDNS:**
    Espera unos minutos. Los pods de Calico y CoreDNS deben pasar a estado `Running`. Puedes verificarlo.

    ```bash
    kubectl get pods -n calico-system
    kubectl get pods -n kube-system
    ```
    [Tu salida de comandos kubectl get pods aquí]

    Si los pods de Calico o CoreDNS se quedan en estado `Pending`, `Init`, o `CrashLoopBackOff`, significa que hay un problema con la red o la CNI (ver sección de Troubleshooting). El error `CoreDNS Pending` es común si la CNI no está instalada o no funciona correctamente.

---