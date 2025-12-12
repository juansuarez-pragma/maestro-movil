# Maestría Empresarial Móvil

## Guía Definitiva de Arquitectura Flutter para Banca y E-commerce

[![License](https://img.shields.io/badge/license-Proprietary-red.svg)]()
[![Flutter](https://img.shields.io/badge/Flutter-3.x-blue.svg)]()
[![Dart](https://img.shields.io/badge/Dart-3.x-blue.svg)]()

---

## Descripción

Este repositorio contiene el contenido del libro **"Maestría Empresarial Móvil"**, una guía exhaustiva de arquitectura Flutter orientada a aplicaciones de **Banca** y **E-commerce** de nivel empresarial.

El contenido está estructurado como **100 casos prácticos** organizados en 10 capítulos, cada uno abordando un escenario real con soluciones arquitectónicas completas.

### Audiencia Objetivo
- Desarrolladores Senior
- Arquitectos de Software
- Equipos técnicos en Banca (Lulo Bank, Bancolombia, NuBank)
- Equipos técnicos en E-commerce (MercadoLibre, Amazon, Rappi)

### Propósito Principal
Este contenido alimenta una **Base de Conocimiento (RAG)** para un Agente de IA que actúa como **"Líder Técnico"**, capaz de:
- Guiar decisiones arquitectónicas basadas en casos reales
- Recomendar patrones según contexto de negocio
- Proporcionar criterios de aceptación técnicos (TACs) por plataforma
- Sugerir estrategias de testing específicas

---

## Estructura del Repositorio

```
maestro-movil/
├── README.md                          # Este archivo
├── CLAUDE.md                          # Contexto para Claude AI
├── TABLA_DE_CONTENIDOS.md             # Índice completo de los 100 casos
├── capitulos/
│   ├── 01-seguridad-bancaria/         # Casos 1-10
│   ├── 02-gestion-estado/             # Casos 11-20
│   ├── 03-rendimiento/                # Casos 21-30
│   ├── 04-offline-first/              # Casos 31-40
│   ├── 05-networking/                 # Casos 41-50
│   ├── 06-arquitectura-modular/       # Casos 51-60
│   ├── 07-integracion-nativa/         # Casos 61-70
│   ├── 08-devops-cicd/                # Casos 71-80
│   ├── 09-hardware-iot/               # Casos 81-90
│   └── 10-migracion-legacy/           # Casos 91-100
└── .gitignore
```

---

## Formato de Cada Caso

Estructura común optimizada para indexación RAG (sin código de implementación):
- **0. Metadata (AI-Tags):** Palabras clave de negocio, patrón técnico, stack, criticidad.
- **1. Planteamiento del Problema:** Problema técnico + escenario de negocio, incidentes reportados y riesgos.
- **2. Matriz de Soluciones:** BAJA / ACEPTABLE / ENTERPRISE con análisis de trade-offs.
- **3. Profundización:** Capacidades vs límites, criterio de selección y tablas de V&V/UX/seguridad/operación cuando apliquen.
- **4. Impacto esperado:** KPIs y resultados de negocio.
- **Glosario de términos clave:** Tabla con ancla `#glosario-de-terminos-clave`; cada término define su `id` (`#term-*`) y el texto del caso debe enlazar a esos términos para habilitar tooltips internos.
- **Referencias:** 3–5 fuentes externas (incidentes, normas, guías).

---

## Stack Tecnológico Cubierto

### Gestión de Estado
| Tecnología | Uso Recomendado |
|:-----------|:----------------|
| **BLoC** | Casos complejos con auditoría de transiciones |
| **Riverpod** | DI moderno, testing con overrides |
| **Cubit** | Casos simples sin eventos complejos |
| **Provider** | Legados, estados mínimos |

### Persistencia
| Tecnología | Uso Recomendado |
|:-----------|:----------------|
| **flutter_secure_storage** | Tokens, credenciales (Keychain/Keystore) |
| **drift** | SQL local con queries complejos |
| **Hive/Isar** | NoSQL rápido, caching |
| **SharedPreferences** | Flags simples, preferencias |

### Networking
| Tecnología | Uso Recomendado |
|:-----------|:----------------|
| **Dio** | HTTP con interceptores, retry, logging |
| **GraphQL** | Queries flexibles, subscriptions |
| **gRPC** | Alta performance, streaming |
| **WebSockets** | Tiempo real, chat |

### Seguridad
| Tecnología | Uso Recomendado |
|:-----------|:----------------|
| **local_auth** | Biometría del dispositivo |
| **Platform Channels** | SDKs nativos (Facetec, Stripe, etc.) |
| **play_integrity** | Attestation Android |
| **App Attest** | Attestation iOS |

---

## Capítulos

| # | Capítulo | Casos | Estado |
|:-:|:---------|:-----:|:------:|
| 1 | Seguridad Bancaria y Gestión de Identidad | 1-10 | ✅ Completado |
| 2 | Gestión de Estado Compleja | 11-20 | ✅ Completado |
| 3 | Optimización de Rendimiento | 21-30 | ✅ Completado |
| 4 | Estrategias Offline-First | 31-40 | ✅ Completado |
| 5 | Networking Avanzado | 41-50 | ✅ Completado |
| 6 | Arquitectura Modular | 51-60 | ✅ Completado |
| 7 | Integración Nativa | 61-70 | ✅ Completado |
| 8 | DevOps y CI/CD | 71-80 | ✅ Completado |
| 9 | Hardware, IoT y Biometría | 81-90 | ✅ Completado |
| 10 | Migración de Legacy | 91-100 | ✅ Completado |

**Leyenda:** ✅ Completado

---

## Uso con Agente IA (RAG)

Este contenido está diseñado para ser consumido por un Agente de IA. Los tags de metadata en cada caso permiten búsqueda semántica precisa:

```
Usuario: "¿Cómo implemento refresh tokens seguros en Flutter?"
Agente: [Busca en metadata: "refresh token", "Token Rotation", "flutter_secure_storage"]
        → Retorna Caso 1 con:
          - Solución ENTERPRISE completa
          - TACs específicos por plataforma (Flutter, Android, iOS)
          - Escenarios de prueba críticos
```

### Ejemplo de Query por Plataforma
```
Usuario: "¿Qué configuración necesito en iOS para certificate pinning?"
Agente: [Busca: "certificate pinning", "iOS", "ATS"]
        → Retorna Caso 3, sección 2.3 iOS y TACs iOS específicos
```

---

## Convenciones del Proyecto

### Nomenclatura de Archivos
- `caso-XX-nombre-descriptivo.md` donde XX es el número del caso
- Nombres en kebab-case, descriptivos del problema

### Referencias a Casos Reales
Cada caso incluye 3–5 referencias a:
- Incidentes de seguridad documentados
- Estadísticas de industria (Verizon, Kaspersky, ACFE)
- Normativas (PCI-DSS, GDPR, PSD2, NIST)

---

## Contribución

Este es un repositorio privado de contenido propietario. Las contribuciones se manejan internamente siguiendo el proceso:

1. Crear rama `feature/caso-XX-descripcion`
2. Seguir el formato de 6 secciones
3. Incluir al menos 2 referencias de industria
4. Separar TACs por plataforma
5. Pull Request con revisión de arquitectura

---

## Licencia

Contenido propietario. Todos los derechos reservados.

---

**Autor:** Arquitectura de Soluciones - Pragma
**Versión:** 1.1.0
**Última actualización:** Diciembre 2024

---

## Changelog

### v1.1.0 (Diciembre 2024)
- Actualizado formato de matriz de soluciones: BAJA/ACEPTABLE/ENTERPRISE
- Añadida sección "Justificación del Plan" en cada caso
- Reorganizada Fase 2 por plataforma (Flutter, Android, iOS)
- TACs ahora separados por plataforma
- Añadida columna Plataforma en escenarios de prueba
- Creado CLAUDE.md para contexto del agente

### v1.0.0 (Diciembre 2024)
- Release inicial con Capítulo 1 completo (10 casos)
- Tabla de contenidos de 100 casos
