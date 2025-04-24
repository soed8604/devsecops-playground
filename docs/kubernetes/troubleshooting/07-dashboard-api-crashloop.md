---
sidebar_position: 7
title: Dashboard API Pod en CrashLoopBackOff
---

# Problema: Dashboard API Pod en CrashLoopBackOff

## Síntoma

Después de instalar el Kubernetes Dashboard con Helm, el pod `kubernetes-dashboard-api-...` en el namespace `kubernetes-dashboard` se queda en estado `CrashLoopBackOff`. Otros pods del Dashboard (auth, web, scraper) pueden estar `Running`.

```bash
kubectl get pods -n kubernetes-dashboard
```
## Diagnóstico
El estado CrashLoopBackOff significa que el contenedor principal dentro del pod arranca y se cierra repetidamente. Revisando los logs del pod se encuentra un error de conexión.
```bash
kubectl logs kubernetes-dashboard-api-<HASH> -n kubernetes-dashboard
```
El log muestra un `i/o timeout` al intentar conectar a `10.96.0.1:443`. Esta es la IP y puerto del servicio ClusterIP del API Server de Kubernetes (`kubernetes.default.svc.cluster.local`). El pod del Dashboard API, al estar dentro del cluster, intenta comunicarse con el API Server usando esta dirección interna.

## Causa Raíz
El `timeout` al intentar conectar al ClusterIP del API Server desde un pod (el del Dashboard API) indica que la red de pods en el nodo donde corre ese pod no está funcionando correctamente o no puede alcanzar el servicio del API Server.

En nuestro caso, el pod del Dashboard API estaba corriendo en `k8s-worker-X`, y el problema era que el agente de red de Calico (`calico-node`) en `k8s-worker-X` no estaba Ready (0/1). Si la CNI no está Ready en un nodo, la red para los pods en ese nodo no funciona, impidiendo la comunicación interna del cluster, incluyendo el acceso a ClusterIPs.

## Solución
Resolver los problemas subyacentes que impiden que el agente de red de Calico (`calico-node`) pase a estado `READY` en el nodo donde corre el pod del Dashboard API.

Las causas comunes de que `calico-node` no esté Ready (0/1) se documentan en detalle en otras secciones de Troubleshooting:
- Enlace a la sección: Calico Node Pod en 0/1 Ready (Overview)
- Enlace a la sección: Calico: BIRD no encuentra archivos de configuración (Problemas de generación de config/confd)
- Enlace a la sección: Calico: Fallo al conectar a Typha (Timeout 5473) (Problemas de conexión a Typha)
- Enlace a la sección: Calico: BGP Peering No Establecido (Puerto 179) (Problemas con puerto 179 y BGP)

Una vez que hayas solucionado el problema específico que impedía a `calico-node` estar `READY` en el nodo worker (probablemente un problema de firewall con el puerto 179 o 5473, o un problema con el pod Typha), la red en ese nodo se corregirá, el pod `kubernetes-dashboard-api` podrá conectar al API Server a través de su ClusterIP, y pasará a estado `Running` (1/1 Ready).

## Verificación

Verifica el estado de los pods del Dashboard nuevamente.

```bash
kubectl get pods -n kubernetes-dashboard
```
Una vez que todos los pods del Dashboard estén `Running` y `Ready`, podrás acceder a la interfaz web (ver sección de Acceso).