1. ¿Quién cambió el RFC de María García en la Actividad 2 y en la Actividad 3?
- En la Actividad 2: No hay evidencia con esta edición. Aunque la bitácora del servidor demuestra que el usuario app_ventas se autenticó a esa hora, el perfilador de base de datos se encontraba apagado (nivel 0), por lo que no quedó registro alguno de qué operación se ejecutó ni qué documento fue modificado.
- En la Actividad 3: Con el perfilador activado en nivel 2, se demuestra que la modificación del RFC a GAMA780912XYZ sobre el documento con identificador 3 fue ejecutada por el usuario app_ventas@comercial.

2. ¿Cuándo y desde qué aplicación?
- Actividad 2: No hay evidencia de la operación en sí (únicamente de la conexión de la aplicación), debido a que el perfilador estaba desactivado al momento del incidente.
- Actividad 3: El cambio se registró el 2026-11-16T15:39:48.727Z (UTC) desde la dirección 127.0.0.1 a través de la aplicación identificada como app-ventas-web.

3. ¿Qué valor tenía antes?
No hay evidencia con esta edición. El perfilador de MongoDB (system.profile) únicamente almacena la instrucción de actualización enviada por el cliente ($set: { rfc: "GAMA780912XYZ" }), pero no guarda el estado previo (before-image) del documento antes de ser alterado ni existen disparadores (triggers) nativos en el motor para capturar dicho valor.

4. ¿Cambió alguien los privilegios de app_ventas?
No hay evidencia nativa con esta edición sobre quién realizó el cambio ni el momento exacto en que ocurrió. MongoDB Community Edition no audita los comandos administrativos de control de acceso (grantRolesToUser ni revokeRolesFromUser) en system.profile ni en la bitácora del servidor. Únicamente mediante la aplicación del control compensatorio de instantáneas periódicas de privilegios se pudo detectar que, entre dos tomas de muestra, se agregó y posteriormente se revocó el rol dbAdmin sobre la base comercial para el usuario app_ventas.