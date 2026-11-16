### 9.1 Interpretación de los documentos

| # | op | Usuario | Cliente | Aplicación | Momento |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | query | app_ventas@comercial | 127.0.0.1 | app-ventas-web | 2026-11-16T15:39:48.724Z |
| 2 | update | app_ventas@comercial | 127.0.0.1 | app-ventas-web | 2026-11-16T15:39:48.727Z |
| 3 | remove | app_ventas@comercial | 127.0.0.1 | app-ventas-web | 2026-11-16T15:39:48.728Z |

---

### Preguntas

1. ¿Puedes saber cuál era el RFC antes del cambio con la información de system.profile?
No es posible saberlo. El perfilador de base de datos registra únicamente el comando enviado al servidor (en este caso, el operador de actualización sobre el documento con identificador 3 asignando el nuevo valor GAMA780912XYZ), pero no almacena el estado previo (before-image) del documento antes de ser modificado.

2. ¿Puedes saber qué datos tenía el documento eliminado?
No es posible saberlo. El documento de perfilado para la operación de borrado (remove) únicamente conserva el criterio o filtro de búsqueda utilizado (el identificador 2) y la cantidad de documentos afectados (ndeleted: 1), sin respaldar el contenido completo del registro eliminado.

3. A diferencia de log_statement = 'mod' en PostgreSQL, el nivel 2 sí registró la consulta. ¿Qué ventaja y qué riesgo tiene eso?
La ventaja principal es la trazabilidad completa de lectura, lo que permite auditar quién consultó información confidencial o sensible y analizar el desempeño de todas las consultas ejecutadas. El riesgo es doble: en el plano operativo, degrada el rendimiento del servidor y llena rápidamente el límite de 1 MB de la colección system.profile sobrescribiendo evidencia previa; en el plano de seguridad y privacidad, expone datos personales o sensibles directamente en texto plano dentro de los filtros de búsqueda registrados en la bitácora del perfilador.

4. ¿Por qué el campo user incluye @comercial?
Porque en MongoDB la identidad de un usuario está vinculada a la base de datos de autenticación (authentication database) en la que fue creado y definido. Dado que el usuario app_ventas fue creado específicamente dentro de la base de datos comercial (a diferencia del usuario admin que reside en admin), su identificador completo en el sistema es app_ventas@comercial.