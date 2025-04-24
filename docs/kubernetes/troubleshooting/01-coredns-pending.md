---
sidebar_position: 1
title: CoreDNS Pods en Pending
---

# Problema: CoreDNS Pods en Pending

## Síntoma

Después de ejecutar `kubeadm init` en el primer nodo Control Plane, los pods de CoreDNS en el namespace `kube-system` se quedan en estado `Pending`.

```bash
kubectl get pods -n kube-system
```
Si describes uno de los pods de CoreDNS (kubectl describe pod `nombre_pod_coredns` -n kube-system), verás eventos indicando FailedScheduling con un mensaje sobre network plugin.

## Diagnóstico
Los pods de CoreDNS (y la mayoría de los pods de usuario) requieren una red de pods funcional proporcionada por una CNI (Container Network Interface) para poder arrancar y comunicarse. Como aún no habíamos instalado la CNI (Calico), Kubernetes no podía programar (schedule) los pods que dependen de ella, como CoreDNS.

## Causa Raíz
Falta de una CNI instalada y funcional en el cluster.

## Solución
Instalar la CNI (Calico en nuestro caso). Este paso se documenta en la sección de Instalación.

```bash
# Comando para instalar Calico
kubectl apply -f calico.yaml
```
## Verificación
Después de aplicar el manifiesto de la CNI y esperar unos minutos, los pods de Calico y CoreDNS deberían pasar a estado Running.

```bash
kubectl get pods -n kube-system
kubectl get pods -n calico-system # Verifica también los pods de Calico
```