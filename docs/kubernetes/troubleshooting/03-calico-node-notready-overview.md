---
sidebar_position: 3
title: Calico Node Pod en 0/1 Ready
---

# Problema: Calico Node Pods en 0/1 Ready

## Síntoma

Después de instalar Calico, observas que el pod `calico-node` en uno o más nodos (particularmente en los nodos Worker como `k8s-worker-02`) se queda en estado `0/1 Running` (Running, pero no Ready).

```bash
kubectl get pods -n calico-system -o wide
```
Esto indica que el agente de red de Calico no ha podido inicializarse completamente en ese nodo. Si `calico-node` no está Ready en un nodo, la red de Pods en ese nodo no funcionará correctamente.

## Diagnóstico Inicial
Cuando un pod está en `0/1 Ready`, lo primero es revisar sus eventos y logs para ver por qué no cumple con la `Readiness probe` o si hay errores al iniciar sus procesos internos.

### Pasos:
1. Obtén los detalles y eventos del pod `calico-node` que está 0/1 Ready:
```bash
kubectl describe pod calico-node-<HASH> -n calico-system
```
2. Obtén los logs del contenedor principal del pod `calico-node`:
```bash
kubectl logs calico-node-<HASH> -n calico-system
```
La revisión de estos logs y eventos reveló varios problemas subyacentes, que documentamos en las siguientes secciones de troubleshooting. La causa del `0/1 Ready` puede variar.
