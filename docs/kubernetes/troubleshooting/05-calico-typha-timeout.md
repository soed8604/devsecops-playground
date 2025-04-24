---
sidebar_position: 5
title: Calico: Fallo al conectar a Typha (Timeout 5473)
---

# Problema: Calico Node no puede conectar a Typha

## Síntoma

Al revisar los logs del pod `calico-node` (o `confd` dentro de él), encuentras advertencias o errores indicando fallos de conexión a un endpoint "typha", a menudo con un `i/o timeout` en el puerto 5473 TCP.

```bash
kubectl logs calico-node-<HASH> -n calico-system # O -c confd
```
## Diagnóstico
El componente `calico-node` (y `confd`) necesita comunicarse con el servicio de Calico llamado `Typha`, que generalmente corre en los nodos Control Plane y ayuda a escalar Calico. El `timeout` al conectar a `IP_del_Master:5473` indica que la conexión TCP no se está estableciendo. Esto impide que `calico-node` obtenga información vital (posiblemente incluyendo datos necesarios para generar la configuración de BIRD).

## Causa Raíz
La causa más probable es que un firewall esté bloqueando el tráfico en el puerto 5473 TCP entre el nodo donde falla `calico-node` (ej: k8s-worker-02) y los nodos Control Plane donde corre `Typha` (ej: k8s-master-01, 02, 03). También podría ser que los pods `calico-typha` no estén corriendo o estén unhealthy.

## Solución
Asegurarse de que el puerto 5473/TCP está abierto en los firewalls de los nodos y que los pods `calico-typha` están saludables.

### Pasos:

1. Verifica el estado de los pods `calico-typha` en los nodos Control Plane:
```bash
kubectl get pods -n calico-system -o wide | grep typha
```
Si no están Running/Ready, investiga esos pods (`kubectl describe/logs`).

2. Verifica el Firewall (UFW) en CADA UNO DE TUS 5 NODOS (maestros y workers). El puerto 5473/TCP debe estar abierto para tráfico entrante en los nodos Control Plane (donde corre Typha) desde los nodos Worker, y saliente desde los nodos Worker.
```bash
sudo ufw status verbose
```
Busca una regla ALLOW para el puerto 5473/tcp. Si no existe, añádela:
```bash
# Si necesitas añadir la regla 5473/tcp en un nodo:
# Conectado por SSH a ese nodo
sudo ufw allow 5473/tcp
sudo ufw reload
```
3. Desde el nodo worker donde falla Calico (ej: k8s-worker-02), prueba la conexión manual a Typha:
```bash
nc -zv 192.168.100.13 5473 # Reemplaza 192.168.100.13 por la IP de un maestro donde corre Typha
```
Debería decir `Connection to ... succeeded!`. Si falla, el problema de conexión persiste.

Una vez que la conexión a Typha sea posible y los pods Typha estén saludables, el pod `calico-node` debería poder inicializarse correctamente y pasar a estado `READY`, lo que permitirá a `confd` generar los archivos de configuración de BIRD.


