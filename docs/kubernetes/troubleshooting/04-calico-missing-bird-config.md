---
sidebar_position: 4
title: "Calico: BIRD no encuentra archivos de configuración"
---

# Problema: Calico/BIRD no encuentra config

## Síntoma

Al revisar los logs del pod `calico-node` (después de ver que está 0/1 Ready), encuentras mensajes de error de BIRD indicando que no puede abrir sus archivos de configuración (`bird.cfg` y `bird6.cfg`).

```bash
kubectl logs calico-node-<HASH> -n calico-system
```
## Diagnóstico
El proceso BIRD, esencial para el enrutamiento BGP en Calico, no puede iniciar porque los archivos de configuración que necesita no existen en la ubicación esperada dentro del contenedor (`/etc/calico/confd/config/`).

## Causa Raíz
Estos archivos de configuración son generados dinámicamente por otro componente dentro del pod `calico-node`, usualmente el contenedor `confd`. Si el contenedor `confd` falla (ej: no puede conectar al API Server, error de configuración), estos archivos no se generan. El error inicial de mismatch de CIDR (si lo tuviste) o problemas de conexión a Typha (ver siguiente sección) pueden impedir que `confd` genere la config correcta.

## Solución
La solución a este síntoma particular de "archivos no encontrados" es resolver la causa raíz por la que el componente `confd` no puede generar los archivos. Esto a menudo implica solucionar problemas de comunicación del pod `calico-node` (y `confd`) con el API Server o con el componente Typha de Calico, o corregir errores en la configuración de Calico aplicada.

Los logs del contenedor `confd` y los logs más detallados del `calico-node` son clave para encontrar la verdadera causa (ver siguientes secciones).

Pasos de diagnóstico para `confd`:

1. Identifica los contenedores en el pod `calico-node`:
```bash
kubectl get pod calico-node-<HASH> -n calico-system -o jsonpath='{.spec.containers[*].name}'
```
2. Obtén los logs del contenedor `confd`:
```bash
kubectl logs calico-node-<HASH> -c confd -n calico-system
```
Estos logs a menudo revelarán problemas de conexión o errores de configuración que impiden la generación de `bird.cfg`.