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
│   ├── 01-seguridad/                  # Casos 1-10
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
- **1. Planteamiento:** Problema técnico + escenario de negocio, incidentes y analítica de prevalencia (tabla) con riesgos.
- **2. Matriz de Soluciones:** BAJA / ACEPTABLE / ENTERPRISE con trade-offs claros.
- **3. Profundización:** Capacidades vs límites, criterio de selección y tablas de V&V, UX/operación/seguridad, mini-ADR (decisión) y observabilidad.
- **4. Impacto esperado:** KPIs con objetivos/umbrales e impacto.
- **Glosario:** Tabla con ancla `#glosario-de-terminos-clave`; cada término define su `id` (`#term-*`) y el texto del caso enlaza a esos términos para tooltips internos.
- **Referencias:** 3–5 fuentes (incidentes, normas, guías, estudios de seguridad como NowSecure).

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
| 1 | Seguridad Bancaria y Gestión de Identidad | 1-10 | ✅ |
| 2 | Gestión de Estado Compleja | 11-20 | ✅ |
| 3 | Optimización de Rendimiento | 21-30 | ✅ |
| 4 | Estrategias Offline-First | 31-40 | ✅ |
| 5 | Networking Avanzado | 41-50 | ✅ |
| 6 | Arquitectura Modular | 51-60 | ✅ |
| 7 | Integración Nativa | 61-70 | ✅ |
| 8 | DevOps y CI/CD | 71-80 | ✅ |
| 9 | Hardware, IoT y Biometría | 81-90 | ✅ |
| 10 | Migración de Legacy | 91-100 | ✅ |

---

## Uso con Agente IA (RAG)

El contenido está diseñado para búsqueda semántica y respuestas estructuradas:

```
Usuario: "¿Cómo manejar refresh tokens rotativos?"
Agente: Busca metadata ("refresh", "rotation", "token_family") → retorna Caso 1 con:
  - Matriz BAJA/ACEPTABLE/ENTERPRISE
  - Tablas de V&V y UX/operación
  - KPIs de impacto (p95 refresh, reuse=0)
```

```
Usuario: "¿Cómo habilitar gRPC en Flutter con fallback?"
Agente: Busca ("gRPC", "fallback", "HTTP/2") → Caso 49 con capacidades/restricciones,
  mini-ADR y KPIs (latencia, disponibilidad).
```

---

## Convenciones del Proyecto

- Archivos `caso-XX-nombre-descriptivo.md` (kebab-case, con cero a la izquierda).
- Cada caso enlaza términos clave a su glosario (`#term-*`) y referencias externas (incidentes, normas, guías, estudios de seguridad/observabilidad).
- Guía editorial y de contribución: ver `AGENTS.md` (formato, anchors de glosario, estilo, commits).

---

## Contribución

Repositorio privado; contribuciones internas siguen:
1. Rama `feature/caso-XX-descripcion`.
2. Formato 0–4 + glosario + referencias (tablas de analítica/V&V/UX/operación donde apliquen).
3. Mínimo 3 referencias de industria/seguridad; anclas de glosario con `#term-*`.
4. PR con revisión técnica/arquitectura.

---

## Licencia

Contenido propietario. Todos los derechos reservados.

---

**Autor:** Arquitectura de Soluciones - Pragma
**Versión:** 1.2.0
**Última actualización:** Enero 2025

---

## Changelog

### v1.2.0 (Enero 2025)
- Todos los 100 casos alineados al formato enriquecido (analítica de industria, tablas de V&V/UX/operación, mini-ADR, KPIs de impacto, glosario con anchors).
- Referencias reforzadas con estudios recientes (NowSecure 2024, guías oficiales).
- Guía de contribución consolidada en `AGENTS.md`.

### v1.1.0 (Diciembre 2024)
- Formato con matriz BAJA/ACEPTABLE/ENTERPRISE y metadata AI-Tags para RAG.
- CLAUDE.md agregado para contexto del agente.
