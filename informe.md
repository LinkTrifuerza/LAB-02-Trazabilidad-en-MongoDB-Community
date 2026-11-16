## 1. Despliegue, edición y limitación

El laboratorio se realizó sobre una instalación nativa de MongoDB Community Edition 7.0.48 en Windows, ejecutándose como servicio local con la autorización habilitada (security.authorization: enabled). Mediante el comando buildInfo se verificó que el arreglo modules está vacío ([]), lo que confirma que se trata de la edición Community.

Esta edición no cuenta con auditoría nativa de seguridad (Native Auditing), la cual está reservada exclusivamente para MongoDB Enterprise y MongoDB Atlas. Por lo tanto, no es posible activar parámetros como --auditDestination ni generar pistas de auditoría dedicadas para eventos administrativos o de seguridad.

## 2. Perfilador

Para rastrear las operaciones sobre la base de datos comercial se utilizó el Database Profiler mediante db.setProfilingLevel(2), lo que permitió registrar en la colección system.profile las consultas (query), actualizaciones (update) y eliminaciones (remove) ejecutadas por el usuario app_ventas@comercial desde la aplicación app-ventas-web.

Sin embargo, el perfilador es una herramienta diseñada para analizar el rendimiento y no un mecanismo de auditoría. Operar en nivel 2 registra absolutamente todas las operaciones, lo que consume recursos de procesador y disco, y llena rápidamente el límite por defecto de 1 MB de system.profile. Para mitigar esto se aplicó un filtro en nivel 1 que conserva únicamente las operaciones de escritura (insert, update y remove), sacrificando la visibilidad de las lecturas a cambio de mayor estabilidad y retención.

## 3. Bitácora de mongod

La bitácora del servidor se guarda en formato JSON en la ruta configurada en systemLog.path y los últimos 1024 eventos pueden consultarse en memoria mediante el comando getLog. Al analizarla, se logró distinguir claramente entre accesos exitosos (result: 0) y fallidos (códigos de error distintos de cero como falla de contraseña o usuario inexistente), además de identificar el nombre de la aplicación cliente.

Asimismo, se comprobó que aunque los cambios de configuración del perfilador ("Profiler settings changed") no incluyen directamente el nombre del usuario, sí registran el identificador de conexión (ctx: "conn44"). Al cruzar ese identificador con el evento previo de autenticación exitosa en la misma conexión, se logró atribuir el cambio al usuario admin.

## 4. Privilegios

Ni el perfilador ni la bitácora del servidor registran los comandos administrativos de asignación o revocación de permisos (grantRolesToUser y revokeRolesFromUser). En MongoDB Community no existe evidencia nativa de quién ni en qué momento exacto modificó los privilegios de un usuario.

Como control compensatorio, se implementó una colección dedicada (auditoria.instantaneas_privilegios) donde se guardan copias periódicas de los usuarios y sus roles. Este mecanismo permite detectar qué permisos cambiaron entre dos tomas de muestra, pero tiene una limitación importante: no sustituye a una auditoría real, ya que no indica quién realizó la modificación ni la hora exacta en que sucedió.

## 5. Conservación

Durante la práctica se identificaron tres riesgos principales de pérdida de información: la sobrescritura automática de registros antiguos cuando system.profile alcanza su límite de 1 MB, el vaciado de la memoria de getLog al reiniciar el servicio, y el restablecimiento del nivel de perfilado a 0 tras un reinicio si no se fija en el archivo de configuración. Para asegurar la integridad de la evidencia antes de reiniciar el servicio, se exportaron los registros a system-profile.json y bitacora-mongod.log, generando sus respectivas firmas SHA-256 en hashes.sha256.

## 6. Comparación

Al comparar los tres escenarios evaluados en la unidad:

- PostgreSQL (con bitácora y disparadores) ofrece el mayor control en entornos gratuitos, logrando capturar accesos, cambios de estructura, privilegios y hasta el valor anterior de los datos modificados.
- MongoDB Community ofrece una trazabilidad parcial: registra accesos en su bitácora y operaciones CRUD si se enciende el perfilador, pero carece de registro de privilegios y no guarda el estado previo de los documentos modificados.
- MongoDB Enterprise y Atlas (según su documentación oficial) cubren de forma nativa los eventos de autenticación, autorización, cambios de esquema y operaciones CRUD mediante auditoría dedicada.

## 7. Recomendación

Para una organización que requiera auditar cambios de privilegios con validez formal y atribución exacta de responsabilidad, se recomienda migrar a MongoDB Enterprise o a un clúster dedicado en MongoDB Atlas (M10 o superior). Como hecho verificado en este laboratorio, MongoDB Community carece de registro nativo para comandos de control de acceso y el control compensatorio de instantáneas solo detecta diferencias entre intervalos sin identificar al autor. Mientras se concreta una eventual migración, se sugiere automatizar la toma de instantáneas de privilegios con una cuenta de servicio de solo inserción y restringir al máximo el acceso administrativo a la base de datos.

---

## Preguntas de reflexión

1. ¿Por qué el perfilador no debe presentarse como un mecanismo de auditoría de seguridad?
   Porque fue diseñado para diagnosticar el desempeño de consultas, no para garantizar seguridad. Puede apagarse o alterarse por un administrador, pierde registros automáticamente al llenarse su límite de 1 MB, no registra cambios de privilegios y no cuenta con garantías de inmutabilidad.
2. ¿Por qué el perfilador sí aporta trazabilidad aunque no sea auditoría?
   Porque cuando está activo permite reconstruir qué operaciones se ejecutaron sobre una colección, en qué momento exacto ocurrieron, desde qué dirección IP, qué aplicación las envió y qué usuario autenticado las realizó.
3. ¿Qué riesgo introduce el nivel 2 al registrar filtros de consultas que pueden contener datos personales?
   Al guardar el comando completo de cada consulta de lectura, cualquier búsqueda filtrada por datos sensibles (como RFC, correo electrónico, teléfono o nombres) queda almacenada en texto plano dentro de system.profile, exponiendo información confidencial a cualquier usuario con permisos para leer esa colección o sus archivos exportados.
4. ¿Qué diferencia hay entre detectar un cambio y atribuirlo a alguien?
   Detectar un cambio significa comprobar que un dato o un permiso es distinto respecto a su estado anterior (como ocurre con las instantáneas de privilegios). Atribuirlo a alguien implica contar con evidencia verificable que vincule esa acción específica con una identidad de usuario, una sesión y un momento exacto.
5. ¿Por qué reconocer una limitación técnica es parte del trabajo profesional del DBA?
   Porque asumir o reportar que una herramienta cubre riesgos que en realidad no controla genera una falsa sensación de seguridad. Documentar con honestidad lo que el motor no puede registrar permite diseñar controles compensatorios adecuados, gestionar los riesgos frente a auditorías y justificar técnicamente futuras inversiones en infraestructura o licenciamiento.
