# Maestría Empresarial Móvil

## Guía Definitiva de Arquitectura Flutter para Banca y E-commerce

[![License](https://img.shields.io/badge/license-Proprietary-red.svg)]()
[![Flutter](https://img.shields.io/badge/Flutter-3.x-blue.svg)]()
[![Dart](https://img.shields.io/badge/Dart-3.x-blue.svg)]()

---

## Descripción

Este repositorio contiene el contenido del libro **"Maestría Empresarial Móvil"**, una guía exhaustiva de arquitectura Flutter orientada a aplicaciones de **Banca** y **E-commerce** de nivel empresarial.

### Audiencia Objetivo
- Desarrolladores Senior
- Arquitectos de Software
- Equipos técnicos en Banca (Lulo Bank, Bancolombia, NuBank)
- Equipos técnicos en E-commerce (MercadoLibre, Amazon, Rappi)

### Propósito
Este contenido alimenta una **Base de Conocimiento (RAG)** para un Agente de IA que actúa como **"Líder Técnico"**, capaz de:
- Guiar decisiones arquitectónicas
- Recomendar patrones según contexto
- Proporcionar criterios de aceptación técnicos
- Sugerir estrategias de testing

---

## Estructura del Repositorio

```
maestro-movil/
├── README.md                          # Este archivo
├── TABLA_DE_CONTENIDOS.md             # Índice completo de los 100 casos
├── capitulos/
│   ├── 01-seguridad-bancaria/         # Casos 1-10
│   │   ├── README.md
│   │   ├── caso-01-token-nunca-expira.md
│   │   ├── caso-02-biometria-falsificada.md
│   │   └── ...
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

Cada caso sigue un formato estandarizado optimizado para indexación RAG:

### 0. Metadata para Indexación (AI-Tags)
- Palabras Clave de Negocio
- Patrón Técnico
- Stack Seleccionado
- Nivel de Criticidad

### 1. Planteamiento del Problema
- Escenario de Negocio (Historia de Usuario)
- Evidencia de Industria (Casos reales documentados)
- Riesgos (Económico, Técnico, Reputacional)

### 2. Matriz de Soluciones
| Rol | Solución | Trade-offs |
|-----|----------|------------|
| Junior | ... | Por qué falla |
| Senior | ... | Mejoras y limitaciones |
| Architect | ... | Solución enterprise |

### 3. Profundización Técnica
- Capacidades (qué SÍ permite)
- Restricciones (qué NO permite)
- Criterios de Selección de herramientas

### 4. Estrategia de Implementación
- Fase 1: Diseño y Arquitectura
- Fase 2: Detalles Técnicos de Implementación
- Fase 3: Observability y Métricas

### 5. Criterios de Aceptación Técnicos (TACs)
Lista de requisitos copiables para Jira/tickets

### 6. Estrategia de Pruebas
- Stack de Testing
- Escenarios Críticos Obligatorios

---

## Stack Tecnológico Cubierto

### Gestión de Estado
- **BLoC** - Casos complejos/auditables
- **Riverpod** - Casos modernos con DI
- **Cubit** - Casos simples
- **Provider** - Legados/simples

### Persistencia
- **Hive/Isar** - NoSQL rápido
- **Drift** - Relacional/SQL
- **Secure Storage** - Tokens y credenciales
- **SharedPreferences** - Flags simples

### Networking
- **Dio** - HTTP con interceptores
- **GraphQL** - Queries flexibles
- **gRPC** - Alta performance
- **WebSockets** - Tiempo real

---

## Capítulos

| # | Capítulo | Casos | Estado |
|:-:|:---------|:-----:|:------:|
| 1 | Seguridad Bancaria y Gestión de Identidad | 1-10 | ✅ |
| 2 | Gestión de Estado Compleja | 11-20 | 🔄 |
| 3 | Optimización de Rendimiento | 21-30 | 📋 |
| 4 | Estrategias Offline-First | 31-40 | 📋 |
| 5 | Networking Avanzado | 41-50 | 📋 |
| 6 | Arquitectura Modular | 51-60 | 📋 |
| 7 | Integración Nativa | 61-70 | 📋 |
| 8 | DevOps y CI/CD | 71-80 | 📋 |
| 9 | Hardware, IoT y Biometría | 81-90 | 📋 |
| 10 | Migración de Legacy | 91-100 | 📋 |

**Leyenda:** ✅ Completado | 🔄 En progreso | 📋 Pendiente

---

## Uso con Agente IA

Este contenido está diseñado para ser consumido por un Agente de IA (RAG). Los tags de metadata en cada caso permiten:

```
Usuario: "¿Cómo implemento refresh tokens seguros?"
Agente: [Busca en metadata: "refresh token", "sesión", "Token Rotation"]
        → Retorna Caso 1 con solución arquitectónica completa
```

---

## Contribución

Este es un repositorio privado de contenido propietario. Las contribuciones se manejan internamente.

---

## Licencia

Contenido propietario. Todos los derechos reservados.

---

**Autor:** Arquitectura de Soluciones - Pragma
**Versión:** 1.0.0
**Última actualización:** Diciembre 2024
