---
sidebar_position: 6
title: Calico: BGP Peering No Establecido (Puerto 179)
---

# Problema: Calico/BIRD no establece BGP Peering

## Síntoma

La `Readiness probe` del pod `calico-node` (vista en `kubectl describe pod ...`) indica que BGP no está establecido con otros nodos.

Readiness probe failed: ... BGP not established with 192.168.100.12,192.168.100.13,192.168.100.14,192.168.100.15

Esto ocurre después de que BIRD ya ha intentado arrancar, lo cual implica que los archivos de configuración existían (o al menos intentó leerlos), pero no puede formar las conexiones de vecino BGP.

## Diagnóstico
El BGP peering entre nodos de Calico se realiza por **TCP en el puerto 179**. Si los nodos no pueden conectar en este puerto, el peering no se establecerá.

## Causa Raíz

El firewall (UFW) en uno o más de los nodos está bloqueando el tráfico entrante en el puerto **179 TCP** desde las IPs de otros nodos de Kubernetes.

## Solución

Asegurarse de que el puerto 179/TCP está abierto en los firewalls de **TODOS** los nodos de Kubernetes (los 3 maestros y los 2 workers).

**Pasos:**

1.  **Verifica el Firewall (UFW) en CADA UNO DE TUS 5 NODOS (maestros y workers).** El puerto **179/TCP** debe estar abierto para tráfico entrante desde las IPs de los otros nodos o desde tu subred de nodos.
```bash
# Conectado por SSH a CADA UNO de los 5 nodos
sudo ufw status verbose
```
Busca una regla `ALLOW` para el puerto `179/tcp`. Si no existe, añádela:
    ```bash
    # Conectado por SSH a un nodo (ej: k8s-worker-02)
    # Prueba la conexión a otro nodo (ej: k8s-master-01)
    nc -zv 192.168.100.12 179 # Reemplaza IPs según necesites probar
    # Prueba la conexión entre varios pares de nodos
    ```
2.  **Desde cualquier nodo, prueba la conexión manual al puerto 179 en otro nodo:**
```bash
# Conectado por SSH a un nodo (ej: k8s-worker-02)
# Prueba la conexión a otro nodo (ej: k8s-master-01)
nc -zv 192.168.100.12 179 # Reemplaza IPs según necesites probar
# Prueba la conexión entre varios pares de nodos
```
Debería decir `Connection to ... succeeded!` si el puerto está abierto.

Una vez que el puerto 179 esté abierto en todos los firewalls y los pods `calico-node` puedan comunicarse por BGP, deberían establecer el peering. Después de unos minutos, los pods `calico-node` deberían pasar a estado `READY` (1/1 Running).

## Verificación Final de Calico

Después de solucionar los problemas de Calico (CIDR, Typha, BGP), todos los pods `calico-node` en todos los nodos deberían estar `1/1 Running`.

```bash
kubectl get pods -n calico-system -o wide
```
Y los nodos Worker deberían pasar a estado `Ready` en `kubectl get nodes`.