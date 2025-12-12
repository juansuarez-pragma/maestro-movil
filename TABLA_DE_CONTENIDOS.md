# Tabla de Contenidos - 100 Casos

## Navegación Rápida

- [Capítulo 1: Seguridad Bancaria](#capítulo-1-seguridad-bancaria-y-gestión-de-identidad)
- [Capítulo 2: Gestión de Estado](#capítulo-2-gestión-de-estado-compleja-y-consistencia-de-datos)
- [Capítulo 3: Rendimiento](#capítulo-3-optimización-de-rendimiento-y-rendering)
- [Capítulo 4: Offline-First](#capítulo-4-estrategias-offline-first-y-sincronización)
- [Capítulo 5: Networking](#capítulo-5-networking-avanzado-e-integración-de-apis)
- [Capítulo 6: Arquitectura Modular](#capítulo-6-arquitectura-modular-y-micro-frontends)
- [Capítulo 7: Integración Nativa](#capítulo-7-integración-nativa-y-platform-channels)
- [Capítulo 8: DevOps](#capítulo-8-devops-cicd-y-estrategia-de-release)
- [Capítulo 9: Hardware e IoT](#capítulo-9-hardware-iot-y-biometría)
- [Capítulo 10: Migración Legacy](#capítulo-10-migración-de-legacy-y-gestión-de-deuda-técnica)

---

## Capítulo 1: Seguridad Bancaria y Gestión de Identidad

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 1 | **El Token que Nunca Expira:** Cómo una Sesión Zombie Costó $2.4M en Fraude | [caso-01](capitulos/01-seguridad-bancaria/caso-01-token-nunca-expira.md) |
| 2 | **Biometría Falsificada:** Cuando Face ID No Es Suficiente para Aprobar un Crédito | [caso-02](capitulos/01-seguridad-bancaria/caso-02-biometria-falsificada.md) |
| 3 | **El Man-in-the-Middle Silencioso:** Certificate Pinning en Transferencias Interbancarias | [caso-03](capitulos/01-seguridad-bancaria/caso-03-mitm-certificate-pinning.md) |
| 4 | **Rooted pero Confiable:** Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos | [caso-04](capitulos/01-seguridad-bancaria/caso-04-root-jailbreak-detection.md) |
| 5 | **Keyloggers en el Teclado Virtual:** Protegiendo PINs en Apps de Banca Móvil | [caso-05](capitulos/01-seguridad-bancaria/caso-05-secure-pin-pad.md) |
| 6 | **Session Hijacking en WiFi Público:** El Caso del Café que Vació Cuentas | [caso-06](capitulos/01-seguridad-bancaria/caso-06-session-hijacking-wifi.md) |
| 7 | **El Dilema del Remember Me:** Persistencia Segura de Credenciales en E-commerce | [caso-07](capitulos/01-seguridad-bancaria/caso-07-remember-me-seguro.md) |
| 8 | **OTP Interceptado:** Implementando MFA Resistente a SIM Swapping | [caso-08](capitulos/01-seguridad-bancaria/caso-08-mfa-sim-swapping.md) |
| 9 | **PCI-DSS en el Bolsillo:** Tokenización de Tarjetas sin Tocar Datos Sensibles | [caso-09](capitulos/01-seguridad-bancaria/caso-09-pci-dss-tokenizacion.md) |
| 10 | **El Empleado Deshonesto:** Auditoría y Trazabilidad de Acciones en Apps Internas | [caso-10](capitulos/01-seguridad-bancaria/caso-10-auditoria-trazabilidad.md) |

---

## Capítulo 2: Gestión de Estado Compleja y Consistencia de Datos

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 11 | **El Carrito Fantasma:** Sincronizar Estado entre 5 Pestañas y 3 Dispositivos | [caso-11](capitulos/02-gestion-estado/caso-11-carrito-fantasma.md) |
| 12 | **Race Condition en el Checkout:** Cuando Dos Hilos Debitan la Misma Cuenta | [caso-12](capitulos/02-gestion-estado/caso-12-race-condition-checkout.md) |
| 13 | **Undo Infinito:** Implementando Historial de Acciones en un Editor de Documentos Financieros | [caso-13](capitulos/02-gestion-estado/caso-13-undo-infinito.md) |
| 14 | **El Formulario de 47 Campos:** Validación Reactiva en Onboarding Bancario | [caso-14](capitulos/02-gestion-estado/caso-14-formulario-47-campos.md) |
| 15 | **Optimistic UI que Mintió:** Revertir Estados Cuando el Backend Rechaza | [caso-15](capitulos/02-gestion-estado/caso-15-optimistic-ui.md) |
| 16 | **Estado Global vs Local:** El Dilema del Saldo en Tiempo Real | [caso-16](capitulos/02-gestion-estado/caso-16-estado-global-local.md) |
| 17 | **Wizard de 12 Pasos:** Persistir Progreso de Solicitud de Crédito Hipotecario | [caso-17](capitulos/02-gestion-estado/caso-17-wizard-12-pasos.md) |
| 18 | **El Dashboard que Colapsó:** Orquestar 15 Streams de Datos Simultáneos | [caso-18](capitulos/02-gestion-estado/caso-18-dashboard-colapso.md) |
| 19 | **Conflicto de Merge en el Cliente:** Resolver Ediciones Concurrentes Offline | [caso-19](capitulos/02-gestion-estado/caso-19-conflicto-merge-cliente.md) |
| 20 | **State Machine Financiera:** Modelar el Ciclo de Vida de una Transferencia SPEI/ACH | [caso-20](capitulos/02-gestion-estado/caso-20-state-machine-financiera.md) |

---

## Capítulo 3: Optimización de Rendimiento y Rendering

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 21 | **La Lista de 100,000 Transacciones:** Scroll Infinito sin Memory Leaks | [caso-21](capitulos/03-rendimiento/caso-21-lista-100k-transacciones.md) |
| 22 | **60 FPS o Muerte:** Animaciones Complejas en Pantallas de Trading | [caso-22](capitulos/03-rendimiento/caso-22-60fps-trading.md) |
| 23 | **El Catálogo Pesado:** Lazy Loading de Imágenes en MercadoLibre Scale | [caso-23](capitulos/03-rendimiento/caso-23-catalogo-pesado.md) |
| 24 | **Build Methods Costosos:** Identificar y Eliminar Rebuilds Innecesarios | [caso-24](capitulos/03-rendimiento/caso-24-build-methods-costosos.md) |
| 25 | **El Widget que Congeló el Hilo Principal:** Cómputo Pesado con Isolates | [caso-25](capitulos/03-rendimiento/caso-25-isolates-computo.md) |
| 26 | **Memoria que No Regresa:** Hunting Memory Leaks en Sesiones Largas | [caso-26](capitulos/03-rendimiento/caso-26-memory-leaks.md) |
| 27 | **Cold Start de 8 Segundos:** Optimizar Tiempo de Arranque en Apps Bancarias | [caso-27](capitulos/03-rendimiento/caso-27-cold-start.md) |
| 28 | **El Shader Compilation Jank:** Pre-warming de Shaders en Producción | [caso-28](capitulos/03-rendimiento/caso-28-shader-jank.md) |
| 29 | **Tree Shaking Agresivo:** Reducir 40% del Bundle Size en Apps Enterprise | [caso-29](capitulos/03-rendimiento/caso-29-tree-shaking.md) |
| 30 | **Battery Drain Silencioso:** Optimizar Background Tasks sin Matar la Batería | [caso-30](capitulos/03-rendimiento/caso-30-battery-drain.md) |

---

## Capítulo 4: Estrategias Offline-First y Sincronización

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 31 | **Vender en el Metro:** Cola de Transacciones Offline para POS Móvil | [caso-31](capitulos/04-offline-first/caso-31-cola-offline-pos.md) |
| 32 | **Conflicto de Inventario:** Dos Vendedores, Un Producto, Cero Stock | [caso-32](capitulos/04-offline-first/caso-32-conflicto-inventario.md) |
| 33 | **El Delta Sync Inteligente:** Sincronizar Solo lo Necesario en Redes 2G | [caso-33](capitulos/04-offline-first/caso-33-delta-sync.md) |
| 34 | **CRDT para Carritos:** Resolución Automática de Conflictos sin Servidor | [caso-34](capitulos/04-offline-first/caso-34-crdt-carritos.md) |
| 35 | **Background Sync Prohibido:** Estrategias cuando iOS Mata tu Proceso | [caso-35](capitulos/04-offline-first/caso-35-background-sync-ios.md) |
| 36 | **Cache Invalidation Hell:** Cuándo Confiar y Cuándo Descartar Datos Locales | [caso-36](capitulos/04-offline-first/caso-36-cache-invalidation.md) |
| 37 | **El Pedido Duplicado:** Idempotencia en Retry de Operaciones Críticas | [caso-37](capitulos/04-offline-first/caso-37-idempotencia-retry.md) |
| 38 | **Version Vectors en Banca:** Tracking de Cambios Distribuidos | [caso-38](capitulos/04-offline-first/caso-38-version-vectors.md) |
| 39 | **Offline Login Seguro:** Autenticar sin Conexión sin Comprometer Seguridad | [caso-39](capitulos/04-offline-first/caso-39-offline-login.md) |
| 40 | **El Sync que Tardó 3 Días:** Estrategia de Reconciliación Masiva Post-Desastre | [caso-40](capitulos/04-offline-first/caso-40-reconciliacion-masiva.md) |

---

## Capítulo 5: Networking Avanzado e Integración de APIs

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 41 | **BFF: El Guardián del Móvil:** Orquestar 7 Microservicios en Una Llamada | [caso-41](capitulos/05-networking/caso-41-bff-guardian.md) |
| 42 | **GraphQL Subscriptions en Trading:** Updates en Tiempo Real de Precios | [caso-42](capitulos/05-networking/caso-42-graphql-subscriptions.md) |
| 43 | **El Timeout que Quebró el Banco:** Configurar Retry Policies Inteligentes | [caso-43](capitulos/05-networking/caso-43-retry-policies.md) |
| 44 | **Circuit Breaker Móvil:** Proteger la App cuando el Backend Agoniza | [caso-44](capitulos/05-networking/caso-44-circuit-breaker.md) |
| 45 | **Request Deduplication:** Evitar Doble Cobro por Doble Tap | [caso-45](capitulos/05-networking/caso-45-request-deduplication.md) |
| 46 | **Cursor vs Offset Pagination:** Navegar Millones de Productos Eficientemente | [caso-46](capitulos/05-networking/caso-46-cursor-offset-pagination.md) |
| 47 | **Rate Limiting Graceful:** Degradar Funcionalidad sin Crashear | [caso-47](capitulos/05-networking/caso-47-rate-limiting.md) |
| 48 | **API Versioning Hell:** Soportar 3 Versiones de API en Producción | [caso-48](capitulos/05-networking/caso-48-api-versioning.md) |
| 49 | **gRPC en Flutter:** Comunicación de Alta Performance para Fintech | [caso-49](capitulos/05-networking/caso-49-grpc-flutter.md) |
| 50 | **WebSocket Resiliente:** Mantener Conexión Viva en Chat de Soporte Bancario | [caso-50](capitulos/05-networking/caso-50-websocket-resiliente.md) |

---

## Capítulo 6: Arquitectura Modular y Micro-frontends

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 51 | **Super App Architecture:** Cómo WeChat y Rappi Integran Mini-Apps | [caso-51](capitulos/06-arquitectura-modular/caso-51-super-app.md) |
| 52 | **Feature Flags en Banca:** Lanzar Funciones a 1% de Usuarios sin Deploy | [caso-52](capitulos/06-arquitectura-modular/caso-52-feature-flags.md) |
| 53 | **El Módulo que Pesaba 40MB:** Lazy Loading de Features por Demanda | [caso-53](capitulos/06-arquitectura-modular/caso-53-lazy-loading-modules.md) |
| 54 | **Dependency Injection at Scale:** GetIt vs Riverpod vs Injectable | [caso-54](capitulos/06-arquitectura-modular/caso-54-dependency-injection.md) |
| 55 | **Comunicación entre Features:** Event Bus vs BLoC-to-BLoC | [caso-55](capitulos/06-arquitectura-modular/caso-55-comunicacion-features.md) |
| 56 | **Shared Kernel:** Extraer Código Común sin Crear un Monolito | [caso-56](capitulos/06-arquitectura-modular/caso-56-shared-kernel.md) |
| 57 | **Plugin Architecture:** Permitir que Terceros Extiendan tu App Bancaria | [caso-57](capitulos/06-arquitectura-modular/caso-57-plugin-architecture.md) |
| 58 | **Mono-Repo con Melos:** Gestionar 15 Packages sin Perder la Cordura | [caso-58](capitulos/06-arquitectura-modular/caso-58-monorepo-melos.md) |
| 59 | **Build Time de 45 Minutos:** Estrategias de Compilación Incremental | [caso-59](capitulos/06-arquitectura-modular/caso-59-build-time.md) |
| 60 | **A/B Testing Arquitectónico:** Servir Diferentes UIs desde el Mismo Código | [caso-60](capitulos/06-arquitectura-modular/caso-60-ab-testing.md) |

---

## Capítulo 7: Integración Nativa y Platform Channels

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 61 | **El SDK Nativo Obligatorio:** Integrar Jumio KYC via Platform Channels | [caso-61](capitulos/07-integracion-nativa/caso-61-sdk-nativo-kyc.md) |
| 62 | **Camera Custom para Cheques:** Captura Optimizada con APIs Nativas | [caso-62](capitulos/07-integracion-nativa/caso-62-camera-cheques.md) |
| 63 | **Background Location Legal:** Tracking de Entregas Cumpliendo Políticas | [caso-63](capitulos/07-integracion-nativa/caso-63-background-location.md) |
| 64 | **Push Notifications Ricas:** Acciones Inline en iOS y Android | [caso-64](capitulos/07-integracion-nativa/caso-64-push-notifications.md) |
| 65 | **Deep Linking Bancario:** Abrir la App en la Pantalla Correcta desde Email | [caso-65](capitulos/07-integracion-nativa/caso-65-deep-linking.md) |
| 66 | **In-App Purchases Auditables:** Suscripciones con Receipt Validation | [caso-66](capitulos/07-integracion-nativa/caso-66-in-app-purchases.md) |
| 67 | **El Sensor que iOS No Expone:** Acceder a Hardware via FFI | [caso-67](capitulos/07-integracion-nativa/caso-67-ffi-hardware.md) |
| 68 | **PiP para Video Banking:** Picture-in-Picture en Videollamadas | [caso-68](capitulos/07-integracion-nativa/caso-68-pip-video.md) |
| 69 | **Widgets Nativos en Home:** Mostrar Saldo en Widget de iOS/Android | [caso-69](capitulos/07-integracion-nativa/caso-69-home-widgets.md) |
| 70 | **Siri Shortcuts para Pagos:** "Hey Siri, Paga mi Tarjeta" | [caso-70](capitulos/07-integracion-nativa/caso-70-siri-shortcuts.md) |

---

## Capítulo 8: DevOps, CI/CD y Estrategia de Release

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 71 | **El Deploy del Viernes:** Por Qué Falló y Cómo Prevenirlo con Feature Flags | [caso-71](capitulos/08-devops-cicd/caso-71-deploy-viernes.md) |
| 72 | **Code Signing Nightmare:** Automatizar Certificados en Equipos de 50 Devs | [caso-72](capitulos/08-devops-cicd/caso-72-code-signing.md) |
| 73 | **Flavors para 5 Bancos:** Una Codebase, Múltiples White-Labels | [caso-73](capitulos/08-devops-cicd/caso-73-flavors-white-label.md) |
| 74 | **El Hotfix de Emergencia:** Rollback Strategy cuando App Store Tarda 24h | [caso-74](capitulos/08-devops-cicd/caso-74-hotfix-rollback.md) |
| 75 | **Coverage del 80%:** Estrategia Realista de Testing en CI/CD | [caso-75](capitulos/08-devops-cicd/caso-75-coverage-80.md) |
| 76 | **Canary Release Móvil:** Lanzar a 5% de Usuarios y Monitorear | [caso-76](capitulos/08-devops-cicd/caso-76-canary-release.md) |
| 77 | **El Build que Rompió Producción:** Implementar Staged Rollouts | [caso-77](capitulos/08-devops-cicd/caso-77-staged-rollouts.md) |
| 78 | **Secrets Management:** No Más API Keys en el Repositorio | [caso-78](capitulos/08-devops-cicd/caso-78-secrets-management.md) |
| 79 | **App Size Budget:** Alertar Cuando el Bundle Supera 100MB | [caso-79](capitulos/08-devops-cicd/caso-79-app-size-budget.md) |
| 80 | **Release Train Quincenal:** Coordinar 4 Squads sin Conflictos | [caso-80](capitulos/08-devops-cicd/caso-80-release-train.md) |

---

## Capítulo 9: Hardware, IoT y Biometría

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 81 | **NFC Tap-to-Pay:** Implementar Pagos Contactless Propietarios | [caso-81](capitulos/09-hardware-iot/caso-81-nfc-tap-to-pay.md) |
| 82 | **BLE para Beacons Bancarios:** Ofertas Personalizadas en Sucursales | [caso-82](capitulos/09-hardware-iot/caso-82-ble-beacons.md) |
| 83 | **QR Dinámico Seguro:** Generar Códigos de Pago con Expiración | [caso-83](capitulos/09-hardware-iot/caso-83-qr-dinamico.md) |
| 84 | **El Wearable que Autoriza:** Apple Watch como Segundo Factor | [caso-84](capitulos/09-hardware-iot/caso-84-wearable-auth.md) |
| 85 | **Liveness Detection:** Evitar Fotos de Fotos en Verificación Facial | [caso-85](capitulos/09-hardware-iot/caso-85-liveness-detection.md) |
| 86 | **Voice Biometrics:** "Mi Voz es mi Contraseña" para Banca Telefónica | [caso-86](capitulos/09-hardware-iot/caso-86-voice-biometrics.md) |
| 87 | **Fingerprint Fallback:** Qué Hacer Cuando el Sensor Falla | [caso-87](capitulos/09-hardware-iot/caso-87-fingerprint-fallback.md) |
| 88 | **Geofencing Inteligente:** Bloquear Transacciones Fuera de Zona | [caso-88](capitulos/09-hardware-iot/caso-88-geofencing.md) |
| 89 | **Acelerómetro Anti-Fraude:** Detectar Comportamiento de Bot | [caso-89](capitulos/09-hardware-iot/caso-89-acelerometro-antifraud.md) |
| 90 | **Thermal Camera Integration:** Detección de Fiebre en Apps de Salud | [caso-90](capitulos/09-hardware-iot/caso-90-thermal-camera.md) |

---

## Capítulo 10: Migración de Legacy y Gestión de Deuda Técnica

| Caso | Título | Archivo |
|:----:|:-------|:--------|
| 91 | **Strangler Fig Pattern:** Migrar App Nativa de 8 Años sin Big Bang | [caso-91](capitulos/10-migracion-legacy/caso-91-strangler-fig.md) |
| 92 | **El Monolito de 500K Líneas:** Estrategia de Modularización Gradual | [caso-92](capitulos/10-migracion-legacy/caso-92-monolito-500k.md) |
| 93 | **Bridge Pattern:** Mantener Kotlin y Flutter en Coexistencia | [caso-93](capitulos/10-migracion-legacy/caso-93-bridge-pattern.md) |
| 94 | **Data Migration Nocturna:** Mover 2M de Usuarios sin Downtime | [caso-94](capitulos/10-migracion-legacy/caso-94-data-migration.md) |
| 95 | **API Compatibility Layer:** Soportar Clientes v1, v2 y v3 Simultáneamente | [caso-95](capitulos/10-migracion-legacy/caso-95-api-compatibility.md) |
| 96 | **Testing de Regresión Legacy:** Crear Tests para Código sin Tests | [caso-96](capitulos/10-migracion-legacy/caso-96-testing-legacy.md) |
| 97 | **El Tercero que Cerró:** Reemplazar SDK Deprecated en 30 Días | [caso-97](capitulos/10-migracion-legacy/caso-97-sdk-deprecated.md) |
| 98 | **Deuda Técnica Cuantificada:** Métricas para Convencer al Negocio | [caso-98](capitulos/10-migracion-legacy/caso-98-deuda-tecnica.md) |
| 99 | **Feature Parity Dashboard:** Tracking de Migración Funcionalidad por Funcionalidad | [caso-99](capitulos/10-migracion-legacy/caso-99-feature-parity.md) |
| 100 | **El Día que Apagamos el Legacy:** Checklist de Decommissioning Seguro | [caso-100](capitulos/10-migracion-legacy/caso-100-decommissioning.md) |

---

## Índice por Patrón Técnico

| Patrón | Casos Relacionados |
|:-------|:-------------------|
| Token Management | 1, 7, 39 |
| Biometric Auth | 2, 5, 85, 86, 87 |
| Certificate Pinning | 3 |
| Device Security | 4, 6 |
| MFA / OTP | 8 |
| PCI-DSS / Tokenization | 9 |
| Audit Trail | 10 |
| State Management | 11-20 |
| Performance | 21-30 |
| Offline-First | 31-40 |
| API Design | 41-50 |
| Modular Architecture | 51-60 |
| Platform Integration | 61-70 |
| CI/CD | 71-80 |
| Hardware/IoT | 81-90 |
| Migration | 91-100 |

---

## Índice por Stack Tecnológico

| Tecnología | Casos Relacionados |
|:-----------|:-------------------|
| **BLoC** | 1, 3, 6, 8, 10, 18, 20 |
| **Cubit** | 2, 5, 7 |
| **Riverpod** | 4, 16 |
| **Provider** | 9, 14 |
| **flutter_secure_storage** | 1, 7, 8 |
| **Dio** | 1, 3, 10, 43, 44 |
| **drift** | 10, 31 |
| **Hive/Isar** | 7, 36 |
| **Platform Channels** | 2, 4, 5, 9, 61-70 |

---

*Última actualización: Diciembre 2024*
