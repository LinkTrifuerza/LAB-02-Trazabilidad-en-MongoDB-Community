# LAB-02 — Trazabilidad en MongoDB Community

Práctica de la Unidad III (Auditoría, trazabilidad y gobierno práctico de datos) enfocada en configurar, consultar y evaluar los mecanismos de registro disponibles en MongoDB Community Edition, identificando sus alcances y limitaciones frente a la auditoría nativa.

## Entorno y despliegue
- Motor de base de datos: MongoDB Community Edition 7.0.48
- Cliente utilizado: mongosh 2.14.2
- Sistema operativo: Windows (instalación nativa como servicio local)
- Configuración de seguridad: Control de acceso habilitado mediante security.authorization: enabled en el archivo de configuración del servicio (mongod.cfg).

## Estructura del repositorio
- evidencias/: Documentos de análisis de las actividades (01 a 10), exportación del perfilador (system-profile.json), bitácora del servidor (bitacora-mongod.log) y firmas de integridad (hashes.sha256).
- scripts/comandos-mongosh.js: Colección de consultas y comandos utilizados durante la práctica.
- informe.md: Reporte técnico final y respuestas de reflexión.