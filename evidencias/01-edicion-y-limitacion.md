1. ¿Qué edición de MongoDB estas usando? ¿Qué evidencia lo demuestra?
Se está utilizando MongoDB Community Edition (versión 7.0.48). La evidencia directa se obtuvo al ejecutar la instrucción db.adminCommand({ buildInfo: 1 }).modules, la cual devolvió un arreglo vacío ([]), lo que confirma la ausencia de los módulos Enterprise. Asimismo, el comando db.serverCmdLineOpts().parsed confirmó que el servicio opera con la autorización de seguridad habilitada (security.authorization: enabled).

2. ¿A dónde escribe tu servidor su bitácora? ¿Cómo lo sabes?
El servidor escribe su bitácora en el archivo de registro configurado localmente en el sistema (indicado en el parámetro systemLog.path de la configuración del servicio). Esto se corrobora mediante la inspección de las opciones de arranque con mongosh y mediante el comando db.adminCommand({ getLog: "global" }), el cual permite consultar en tiempo real los eventos recientes almacenados en la memoria del servidor.

3. Según REF-U3-04, ¿qué cuatro categorías de eventos registraría la auditoría nativa?
De acuerdo con la documentación oficial de MongoDB para la auditoría nativa (exclusiva de las ediciones Enterprise y Atlas), las cuatro categorías de eventos que se pueden registrar son:
- Cambios de esquema (DDL).
- Eventos de conjuntos de réplicas y clústeres fragmentados.
- Autenticación y autorización.
- Operaciones CRUD (requiriendo la habilitación de auditAuthorizationSuccess).

4. ¿Qué tendría que evaluar una organización antes de licenciar Enterprise o migrar a Atlas solo para obtener auditoría?
Una organización debe evaluar el costo económico de las licencias o del servicio en la nube frente al valor crítico de los datos gestionados. Esto incluye analizar el cumplimiento normativo estricto requerido por regulaciones externas, el impacto operativo y de rendimiento que introduce la auditoría detallada de operaciones CRUD, y si es viable implementar controles compensatorios internos (como el Database Profiler en nivel 2, el control de acceso estricto RBAC e instantáneas periódicas de privilegios) para mitigar los riesgos de trazabilidad sin incurrir en costos adicionales.