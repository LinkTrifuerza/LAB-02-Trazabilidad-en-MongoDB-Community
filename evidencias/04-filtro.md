1. ¿Qué perdiste de evidencia al aplicar el filtro? ¿Qué ganaste?
   Se perdió la trazabilidad de todas las operaciones de lectura (query), comandos administrativos y consultas que no modifican datos, por lo que ya no es posible auditar qué usuarios consultaron información confidencial. A cambio, se ganó una reducción considerable en el volumen de escritura hacia system.profile, un menor impacto en el rendimiento del servidor (I/O y CPU) y una mayor ventana de retención histórica dentro del límite de 1 MB de la colección limitada, evitando que las lecturas frecuentes sobrescriban las modificaciones críticas.
2. ¿Qué filtro usarías si solo interesaran operaciones sobre la colección clientes?
   Se utilizaría una expresión de filtro evaluando el espacio de nombres (ns) correspondiente a la base de datos y la colección:
   db.setProfilingLevel(1, { filter: { ns: "comercial.clientes" } })
