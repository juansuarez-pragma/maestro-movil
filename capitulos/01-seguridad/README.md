# Capítulo 1: Seguridad

## Resumen del Capítulo

Este capítulo aborda 10 escenarios críticos de seguridad para apps móviles (banca y e-commerce). Cada caso presenta un problema realista, evidencia de industria y una solución arquitectónica en niveles **BAJA/ACEPTABLE/ENTERPRISE**.

## Casos Incluidos

| # | Caso | Criticidad | Stack Principal |
|:-:|:-----|:-----------|:----------------|
| 1 | [Token que Nunca Expira](caso-01-token-nunca-expira.md) | Alta | BLoC + Secure Storage |
| 2 | [Biometría Falsificada](caso-02-biometria-falsificada.md) | Alta | Cubit + Platform Channels |
| 3 | [Certificate Pinning](caso-03-mitm-certificate-pinning.md) | Alta | Dio + BLoC |
| 4 | [Root/Jailbreak Detection](caso-04-root-jailbreak-detection.md) | Alta | Riverpod + Play Integrity |
| 5 | [Secure PIN Pad](caso-05-secure-pin-pad.md) | Alta | Custom Widget + Provider |
| 6 | [Session Hijacking WiFi](caso-06-session-hijacking-wifi.md) | Alta | BLoC + Connectivity |
| 7 | [Remember Me Seguro](caso-07-remember-me-seguro.md) | Media | Cubit + Secure Storage |
| 8 | [MFA vs SIM Swapping](caso-08-mfa-sim-swapping.md) | Alta | BLoC + TOTP |
| 9 | [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md) | Alta | Platform Channels + Provider |
| 10 | [Auditoría y Trazabilidad](caso-10-auditoria-trazabilidad.md) | Alta | BLoC + Drift |

## Patrones Técnicos Cubiertos

- Token Rotation
- Sliding Session
- Secure Token Storage
- Multi-Layer Biometric Authentication
- Step-up Authentication
- Liveness Detection
- Certificate Pinning
- Public Key Pinning
- Device Attestation
- Risk-Based Authentication
- Secure Input
- Network Security Detection
- TOTP / Push MFA
- PCI Scope Reduction
- Audit Trail
- Event Sourcing

## Regulaciones Aplicables

- **PCI-DSS:** Casos 1, 5, 9
- **PSD2/SCA:** Casos 2, 8
- **GDPR:** Caso 10
- **SOX:** Caso 10
- **Regulaciones bancarias locales:** Todos

## Fuentes y literatura base (referencias del capítulo)

- OWASP MASVS/MAS: https://mas.owasp.org/
- OWASP MSTG: https://owasp.org/www-project-mobile-security-testing-guide/
- NIST (autenticación/identidad): https://pages.nist.gov/800-63-3/
- NowSecure 2024 (seguridad móvil): https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/

## Prerequisitos

Para implementar las soluciones de este capítulo, se requiere conocimiento de:

- Clean Architecture en Flutter
- Patrón BLoC/Cubit
- Platform Channels (iOS/Android)
- Criptografía básica (hashing, firmas)
- APIs de seguridad nativas (Keychain, Keystore)

---

*Siguiente capítulo: [Gestión de Estado Compleja](../02-gestion-estado/README.md)*
