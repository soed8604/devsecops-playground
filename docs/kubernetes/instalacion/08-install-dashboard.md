---
sidebar_position: 8
title: Instalar Kubernetes Dashboard
---

# Instalar Kubernetes Dashboard

El Dashboard proporciona una interfaz web gráfica para interactuar y visualizar recursos en tu cluster. No viene instalado por defecto y la forma recomendada de instalarlo es usando Helm.

**Pasos en tu máquina local (donde tienes kubectl y helm instalados):**

1.  **Añade el repositorio oficial de charts del Dashboard a Helm:**

    ```bash
    helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
    ```

2.  **Actualiza tus repositorios de Helm:**

    ```bash
    helm repo update
    ```

3.  **Despliega el Dashboard usando Helm.**
    Este comando instala el chart del Dashboard en el namespace `kubernetes-dashboard`.

    ```bash
    helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
    ```
    [Tu salida del comando helm upgrade --install aquí]

4.  **Verifica que los pods del Dashboard estén corriendo:**
    Espera unos minutos. Los pods en el namespace `kubernetes-dashboard` deberían pasar a estado `Running`.

    ```bash
    kubectl get pods -n kubernetes-dashboard
    ```
    [Tu salida de kubectl get pods -n kubernetes-dashboard aquí]

    Si el pod `kubernetes-dashboard-api` se queda en `CrashLoopBackOff` o algún otro pod falla, consulta la sección de Troubleshooting.

Una vez que todos los pods estén `Running`, puedes acceder al Dashboard (ver sección de Acceso).

---