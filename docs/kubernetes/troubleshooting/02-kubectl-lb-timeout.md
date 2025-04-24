---
sidebar_position: 2
title: Acceso a kubectl desde Local falla (Timeout a LB)
---

# Problema: Acceso a kubectl desde Local falla (Timeout)

## Síntoma

Después de configurar `kubectl` en tu máquina local (copiando `admin.conf`), los comandos como `kubectl get nodes` o `k get pods` fallan con un error `i/o timeout` al intentar conectar a la IP de tu balanceador de carga (ej: `192.168.100.7:6443`).

```bash
k get pods -n kube-system
```
Observaste que al ver los logs de HAProxy (`sudo journalctl -u haproxy -f` en el LB) mientras intentabas conectar desde local, no aparecía ninguna solicitud de conexión.

## Diagnóstico
El hecho de que la solicitud no llegara a los logs de HAProxy indica que el tráfico se estaba bloqueando antes de ser aceptado por el proceso de HAProxy. Dado que `ping` al balanceador funcionaba (conectividad de red básica), el sospechoso principal era el firewall en el servidor del balanceador.

## Causa Raíz
El firewall (UFW) en el servidor del balanceador (192.168.100.7) tenía una regla conflictiva (`DENY IN Anywhere`) que bloqueaba el tráfico entrante en el puerto 6443 TCP, impidiendo que las conexiones desde tu máquina local llegaran a HAProxy.

## Solución
Eliminar la regla DENY conflictiva en el firewall del servidor del balanceador y asegurarse de que la regla ALLOW para tu IP local o subred esté activa para el puerto 6443.

### Pasos en el servidor del balanceador (192.168.100.7):

1. Verifica el estado de UFW (notaste una regla `6443 DENY IN Anywhere`):
    ```bash
    sudo ufw status verbose
    ```
2. Elimina la regla DENY conflictiva (la forma más segura es por número si la tienes listada así con `ufw status numbered`):
     ```bash
    # Primero, lista con números para encontrar el número de la regla "6443 DENY IN Anywhere"
    sudo ufw status numbered
    # Luego, elimina por su número (reemplaza <NUMERO_DE_LA_REGLA_DENY>)
    sudo ufw delete <NUMERO_DE_LA_REGLA_DENY>
    ```
3. Recarga UFW para aplicar los cambios:
    ```bash
    sudo ufw reload
    ```
4. Verifica nuevamente el estado para confirmar que la regla DENY se fue:
    ```bash
    sudo ufw status verbose
    ```
## Verificación
Después de eliminar la regla DENY y recargar UFW, vuelve a tu máquina local e intenta el comando `kubectl get nodes` (o `k get nodes`) de nuevo. Ahora debería poder conectar al balanceador y mostrar el estado de tus nodos.
    ```bash
    kubectl get nodes
    ```

