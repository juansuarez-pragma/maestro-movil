# Tabla de Contenidos - 100 Casos

## Navegación Rápida

- [Capítulo 1: Seguridad Bancaria y Gestión de Identidad](#capitulo-1:-seguridad-bancaria-y-gestion-de-identidad)
- [Capítulo 2: Gestión de Estado Compleja y Consistencia de Datos](#capitulo-2:-gestion-de-estado-compleja-y-consistencia-de-datos)
- [Capítulo 3: Optimización de Rendimiento y Rendering](#capitulo-3:-optimizacion-de-rendimiento-y-rendering)
- [Capítulo 4: Estrategias Offline-First y Sincronización](#capitulo-4:-estrategias-offline-first-y-sincronizacion)
- [Capítulo 5: Networking Avanzado e Integración de APIs](#capitulo-5:-networking-avanzado-e-integracion-de-apis)
- [Capítulo 6: Arquitectura Modular y Micro-frontends](#capitulo-6:-arquitectura-modular-y-micro-frontends)
- [Capítulo 7: Integración Nativa y Platform Channels](#capitulo-7:-integracion-nativa-y-platform-channels)
- [Capítulo 8: DevOps, CI/CD y Estrategia de Release](#capitulo-8:-devops,-cicd-y-estrategia-de-release)
- [Capítulo 9: Hardware, IoT y Biometría](#capitulo-9:-hardware,-iot-y-biometria)
- [Capítulo 10: Migración de Legacy y Gestión de Deuda Técnica](#capitulo-10:-migracion-de-legacy-y-gestion-de-deuda-tecnica)

---
## Capítulo 1: Seguridad Bancaria y Gestión de Identidad

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 1 | **Cómo una Sesión Zombie Costó $2.4M en Fraude** | [caso-01-token-nunca-expira](capitulos/01-seguridad/caso-01-token-nunca-expira.md) |
| 2 | **Cuando Face ID No Es Suficiente para Aprobar un Crédito** | [caso-02-biometria-falsificada](capitulos/01-seguridad/caso-02-biometria-falsificada.md) |
| 3 | **Certificate Pinning en Transferencias Interbancarias** | [caso-03-mitm-certificate-pinning](capitulos/01-seguridad/caso-03-mitm-certificate-pinning.md) |
| 4 | **Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos** | [caso-04-root-jailbreak-detection](capitulos/01-seguridad/caso-04-root-jailbreak-detection.md) |
| 5 | **Protegiendo PINs en Apps de Banca Móvil** | [caso-05-secure-pin-pad](capitulos/01-seguridad/caso-05-secure-pin-pad.md) |
| 6 | **El Caso del Café que Vació Cuentas** | [caso-06-session-hijacking-wifi](capitulos/01-seguridad/caso-06-session-hijacking-wifi.md) |
| 7 | **Persistencia Segura de Credenciales en E-commerce** | [caso-07-remember-me-seguro](capitulos/01-seguridad/caso-07-remember-me-seguro.md) |
| 8 | **Implementando MFA Resistente a SIM Swapping** | [caso-08-mfa-sim-swapping](capitulos/01-seguridad/caso-08-mfa-sim-swapping.md) |
| 9 | **Tokenización de Tarjetas sin Tocar Datos Sensibles** | [caso-09-pci-dss-tokenizacion](capitulos/01-seguridad/caso-09-pci-dss-tokenizacion.md) |
| 10 | **Auditoría y Trazabilidad de Acciones en Apps Internas** | [caso-10-auditoria-trazabilidad](capitulos/01-seguridad/caso-10-auditoria-trazabilidad.md) |

---

## Capítulo 2: Gestión de Estado Compleja y Consistencia de Datos

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 11 | **Sincronizar Estado entre 5 Pestañas y 3 Dispositivos** | [caso-11-carrito-fantasma](capitulos/02-gestion-estado/caso-11-carrito-fantasma.md) |
| 12 | **Cuando Dos Hilos Debitan la Misma Cuenta** | [caso-12-race-condition-checkout](capitulos/02-gestion-estado/caso-12-race-condition-checkout.md) |
| 13 | **Historial de Acciones en un Editor de Documentos Financieros** | [caso-13-undo-infinito](capitulos/02-gestion-estado/caso-13-undo-infinito.md) |
| 14 | **Validación Reactiva en Onboarding Bancario** | [caso-14-formulario-47-campos](capitulos/02-gestion-estado/caso-14-formulario-47-campos.md) |
| 15 | **Revertir Estados Cuando el Backend Rechaza** | [caso-15-optimistic-ui](capitulos/02-gestion-estado/caso-15-optimistic-ui.md) |
| 16 | **El Dilema del Saldo en Tiempo Real** | [caso-16-estado-global-local](capitulos/02-gestion-estado/caso-16-estado-global-local.md) |
| 17 | **Persistir Progreso de Solicitud de Crédito Hipotecario** | [caso-17-wizard-12-pasos](capitulos/02-gestion-estado/caso-17-wizard-12-pasos.md) |
| 18 | **Orquestar 15 Streams de Datos Simultáneos** | [caso-18-dashboard-colapso](capitulos/02-gestion-estado/caso-18-dashboard-colapso.md) |
| 19 | **Resolver Ediciones Concurrentes Offline** | [caso-19-conflicto-merge-cliente](capitulos/02-gestion-estado/caso-19-conflicto-merge-cliente.md) |
| 20 | **Modelar el Ciclo de Vida de una Transferencia SPEI/ACH** | [caso-20-state-machine-financiera](capitulos/02-gestion-estado/caso-20-state-machine-financiera.md) |

---

## Capítulo 3: Optimización de Rendimiento y Rendering

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 21 | **Scroll Infinito sin Memory Leaks** | [caso-21-lista-100k-transacciones](capitulos/03-rendimiento/caso-21-lista-100k-transacciones.md) |
| 22 | **Animaciones Complejas en Pantallas de Trading** | [caso-22-60fps-trading](capitulos/03-rendimiento/caso-22-60fps-trading.md) |
| 23 | **Lazy Loading de Imágenes en MercadoLibre Scale** | [caso-23-catalogo-pesado](capitulos/03-rendimiento/caso-23-catalogo-pesado.md) |
| 24 | **Identificar y Eliminar Rebuilds Innecesarios** | [caso-24-build-methods-costosos](capitulos/03-rendimiento/caso-24-build-methods-costosos.md) |
| 25 | **Cómputo Pesado con Isolates** | [caso-25-isolates-computo](capitulos/03-rendimiento/caso-25-isolates-computo.md) |
| 26 | **Hunting Memory Leaks en Sesiones Largas** | [caso-26-memory-leaks](capitulos/03-rendimiento/caso-26-memory-leaks.md) |
| 27 | **Optimizar Tiempo de Arranque en Apps Bancarias** | [caso-27-cold-start](capitulos/03-rendimiento/caso-27-cold-start.md) |
| 28 | **Pre-warming de Shaders en Producción** | [caso-28-shader-jank](capitulos/03-rendimiento/caso-28-shader-jank.md) |
| 29 | **Reducir 40% del Bundle Size en Apps Enterprise** | [caso-29-tree-shaking](capitulos/03-rendimiento/caso-29-tree-shaking.md) |
| 30 | **Optimizar Background Tasks sin Matar la Batería** | [caso-30-battery-drain](capitulos/03-rendimiento/caso-30-battery-drain.md) |

---

## Capítulo 4: Estrategias Offline-First y Sincronización

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 31 | **Cola de Transacciones Offline para POS Móvil** | [caso-31-cola-offline-pos](capitulos/04-offline-first/caso-31-cola-offline-pos.md) |
| 32 | **Dos Vendedores, Un Producto, Cero Stock** | [caso-32-conflicto-inventario](capitulos/04-offline-first/caso-32-conflicto-inventario.md) |
| 33 | **Sincronizar Solo lo Necesario en Redes 2G** | [caso-33-delta-sync](capitulos/04-offline-first/caso-33-delta-sync.md) |
| 34 | **Resolución Automática de Conflictos sin Servidor** | [caso-34-crdt-carritos](capitulos/04-offline-first/caso-34-crdt-carritos.md) |
| 35 | **Estrategias cuando iOS Mata tu Proceso** | [caso-35-background-sync-ios](capitulos/04-offline-first/caso-35-background-sync-ios.md) |
| 36 | **Cuándo Confiar y Cuándo Descartar Datos Locales** | [caso-36-cache-invalidation](capitulos/04-offline-first/caso-36-cache-invalidation.md) |
| 37 | **Idempotencia en Retry de Operaciones Críticas** | [caso-37-idempotencia-retry](capitulos/04-offline-first/caso-37-idempotencia-retry.md) |
| 38 | **Tracking de Cambios Distribuidos** | [caso-38-version-vectors](capitulos/04-offline-first/caso-38-version-vectors.md) |
| 39 | **Evitar Bloquear la Operación con Syncs Largos** | [caso-39-background-sync](capitulos/04-offline-first/caso-39-background-sync.md) |
| 40 | **Limpiar Datos Huérfanos en Sincronización Offline** | [caso-40-merge-cliente](capitulos/04-offline-first/caso-40-merge-cliente.md) |

---

## Capítulo 5: Networking Avanzado e Integración de APIs

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 41 | **Orquestar 7 Microservicios en Una Llamada** | [caso-41-bff-guardian](capitulos/05-networking/caso-41-bff-guardian.md) |
| 42 | **Updates en Tiempo Real de Precios** | [caso-42-graphql-subscriptions](capitulos/05-networking/caso-42-graphql-subscriptions.md) |
| 43 | **Configurar Retry Policies Inteligentes** | [caso-43-retry-policies](capitulos/05-networking/caso-43-retry-policies.md) |
| 44 | **Proteger la App cuando el Backend Agoniza** | [caso-44-circuit-breaker](capitulos/05-networking/caso-44-circuit-breaker.md) |
| 45 | **Evitar Doble Cobro por Doble Tap** | [caso-45-request-deduplication](capitulos/05-networking/caso-45-request-deduplication.md) |
| 46 | **Navegar Millones de Productos Eficientemente** | [caso-46-cursor-offset-pagination](capitulos/05-networking/caso-46-cursor-offset-pagination.md) |
| 47 | **Degradar Funcionalidad sin Crashear** | [caso-47-rate-limiting](capitulos/05-networking/caso-47-rate-limiting.md) |
| 48 | **Soportar 3 Versiones de API en Producción** | [caso-48-api-versioning](capitulos/05-networking/caso-48-api-versioning.md) |
| 49 | **Comunicación de Alta Performance para Fintech** | [caso-49-grpc-flutter](capitulos/05-networking/caso-49-grpc-flutter.md) |
| 50 | **Mantener Conexión Viva en Chat de Soporte Bancario** | [caso-50-websocket-resiliente](capitulos/05-networking/caso-50-websocket-resiliente.md) |

---

## Capítulo 6: Arquitectura Modular y Micro-frontends

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 51 | **Cómo WeChat y Rappi Integran Mini-Apps** | [caso-51-super-app](capitulos/06-arquitectura-modular/caso-51-super-app.md) |
| 52 | **Lanzar Funciones a 1% de Usuarios sin Deploy** | [caso-52-feature-flags](capitulos/06-arquitectura-modular/caso-52-feature-flags.md) |
| 53 | **Lazy Loading de Features por Demanda** | [caso-53-lazy-loading-modules](capitulos/06-arquitectura-modular/caso-53-lazy-loading-modules.md) |
| 54 | **GetIt vs Riverpod vs Injectable** | [caso-54-dependency-injection](capitulos/06-arquitectura-modular/caso-54-dependency-injection.md) |
| 55 | **Event Bus vs BLoC-to-BLoC** | [caso-55-comunicacion-features](capitulos/06-arquitectura-modular/caso-55-comunicacion-features.md) |
| 56 | **Extraer Código Común sin Crear un Monolito** | [caso-56-shared-kernel](capitulos/06-arquitectura-modular/caso-56-shared-kernel.md) |
| 57 | **Permitir que Terceros Extiendan tu App Bancaria** | [caso-57-plugin-architecture](capitulos/06-arquitectura-modular/caso-57-plugin-architecture.md) |
| 58 | **Gestionar 15 Packages sin Perder la Cordura** | [caso-58-monorepo-melos](capitulos/06-arquitectura-modular/caso-58-monorepo-melos.md) |
| 59 | **Estrategias de Compilación Incremental** | [caso-59-build-time](capitulos/06-arquitectura-modular/caso-59-build-time.md) |
| 60 | **Servir Diferentes UIs desde el Mismo Código** | [caso-60-ab-testing](capitulos/06-arquitectura-modular/caso-60-ab-testing.md) |

---

## Capítulo 7: Integración Nativa y Platform Channels

> Nota: en este repositorio aún no están publicados los casos **67–70**.

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 61 | **Integrar SDKs Nativos (Stripe/Facetec) en Flutter** | [caso-61-platform-channels](capitulos/07-integracion-nativa/caso-61-platform-channels.md) |
| 62 | **Integrar SDKs con Keys, Obfuscation y Auditoría** | [caso-62-sdk-privativo](capitulos/07-integracion-nativa/caso-62-sdk-privativo.md) |
| 63 | **Migrar un Plugin Obsoleto sin Romper Producción** | [caso-63-plugin-nativo-legacy](capitulos/07-integracion-nativa/caso-63-plugin-nativo-legacy.md) |
| 64 | **Autenticación Sin Contraseña con WebAuthn** | [caso-64-fido2-passkeys](capitulos/07-integracion-nativa/caso-64-fido2-passkeys.md) |
| 65 | **Uso de Face/Touch/Voice con SDKs Nativos** | [caso-65-sensor-biometrics](capitulos/07-integracion-nativa/caso-65-sensor-biometrics.md) |
| 66 | **Integrar WebViews/Views Nativos sin Romper la UX** | [caso-66-sdks-pagos-nativos](capitulos/07-integracion-nativa/caso-66-sdks-pagos-nativos.md) |

---

## Capítulo 8: DevOps, CI/CD y Estrategia de Release

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 71 | **Pipelines Reproducibles para Flutter en Banca** | [caso-71-ci-cd-mobile](capitulos/08-devops-cicd/caso-71-ci-cd-mobile.md) |
| 72 | **Gestionar Keystores y Provisioning Profiles con Seguridad** | [caso-72-fastlane-cert](capitulos/08-devops-cicd/caso-72-fastlane-cert.md) |
| 73 | **Calendario de Releases Predecible en Banca** | [caso-73-release-train](capitulos/08-devops-cicd/caso-73-release-train.md) |
| 74 | **Controlar Impacto sin Retrasar Deploys** | [caso-74-feature-flags-release](capitulos/08-devops-cicd/caso-74-feature-flags-release.md) |
| 75 | **Detectar Reversiones Antes de Impactar a Todos** | [caso-75-canary-monitoring](capitulos/08-devops-cicd/caso-75-canary-monitoring.md) |
| 76 | **Logs, Métricas y Trazas en Banca Flutter** | [caso-76-observabilidad-mobile](capitulos/08-devops-cicd/caso-76-observabilidad-mobile.md) |
| 77 | **Medir Uso de Features sin Duplicar Métricas** | [caso-77-feature-analytics](capitulos/08-devops-cicd/caso-77-feature-analytics.md) |
| 78 | **Desplegar en Producción sin Activar la Funcionalidad** | [caso-78-dark-launch](capitulos/08-devops-cicd/caso-78-dark-launch.md) |
| 79 | **Alertar Cuando el Bundle Supera 100MB** | [caso-79-app-size-budget](capitulos/08-devops-cicd/caso-79-app-size-budget.md) |
| 80 | **Coordinar 4 Squads sin Conflictos** | [caso-80-release-train](capitulos/08-devops-cicd/caso-80-release-train.md) |

---

## Capítulo 9: Hardware, IoT y Biometría

> Nota: en este repositorio aún no está publicado el caso **90**.

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 81 | **Sincronizar Dispositivos sin Exponer Datos** | [caso-81-ble-seguro](capitulos/09-hardware-iot/caso-81-ble-seguro.md) |
| 82 | **Leer/Escribir Tags y Tokens sin Riesgo** | [caso-82-nfc-wallet](capitulos/09-hardware-iot/caso-82-nfc-wallet.md) |
| 83 | **Enviar Datos de Dispositivos con Calidad y Seguridad** | [caso-83-iot-telemetria](capitulos/09-hardware-iot/caso-83-iot-telemetria.md) |
| 84 | **Apple Watch como Segundo Factor** | [caso-84-wearable-auth](capitulos/09-hardware-iot/caso-84-wearable-auth.md) |
| 85 | **Evitar Fotos de Fotos en Verificación Facial** | [caso-85-liveness-detection](capitulos/09-hardware-iot/caso-85-liveness-detection.md) |
| 86 | **"Mi Voz es mi Contraseña" para Banca Telefónica** | [caso-86-voice-biometrics](capitulos/09-hardware-iot/caso-86-voice-biometrics.md) |
| 87 | **Qué Hacer Cuando el Sensor Falla** | [caso-87-fingerprint-fallback](capitulos/09-hardware-iot/caso-87-fingerprint-fallback.md) |
| 88 | **Bloquear Transacciones Fuera de Zona** | [caso-88-geofencing](capitulos/09-hardware-iot/caso-88-geofencing.md) |
| 89 | **Detectar Comportamiento de Bot con Sensores** | [caso-89-acelerometro-antifraud](capitulos/09-hardware-iot/caso-89-acelerometro-antifraud.md) |

---

## Capítulo 10: Migración de Legacy y Gestión de Deuda Técnica

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 91 | **Desacoplar sin Detener Releases** | [caso-91-monolito-flutter](capitulos/10-migracion-legacy/caso-91-monolito-flutter.md) |
| 92 | **Evolucionar sin Romper Clientes Antiguos** | [caso-92-api-v1-legacy](capitulos/10-migracion-legacy/caso-92-api-v1-legacy.md) |
| 93 | **Separar en Múltiples Repos sin Perder Trazabilidad** | [caso-93-mono-repo-split](capitulos/10-migracion-legacy/caso-93-mono-repo-split.md) |
| 94 | **Mover 2M de Usuarios sin Downtime** | [caso-94-data-migration](capitulos/10-migracion-legacy/caso-94-data-migration.md) |
| 95 | **Soportar Clientes v1, v2 y v3 Simultáneamente** | [caso-95-api-compatibility](capitulos/10-migracion-legacy/caso-95-api-compatibility.md) |
| 96 | **Crear Tests para Código sin Cobertura** | [caso-96-testing-legacy](capitulos/10-migracion-legacy/caso-96-testing-legacy.md) |
| 97 | **Reemplazar SDK Deprecated en 30 Días** | [caso-97-sdk-deprecated](capitulos/10-migracion-legacy/caso-97-sdk-deprecated.md) |
| 98 | **Métricas para Convencer al Negocio** | [caso-98-deuda-tecnica](capitulos/10-migracion-legacy/caso-98-deuda-tecnica.md) |
| 99 | **Tracking de Migración Funcionalidad por Funcionalidad** | [caso-99-feature-parity](capitulos/10-migracion-legacy/caso-99-feature-parity.md) |
| 100 | **Checklist de Decommissioning Seguro** | [caso-100-decommissioning](capitulos/10-migracion-legacy/caso-100-decommissioning.md) |

---
