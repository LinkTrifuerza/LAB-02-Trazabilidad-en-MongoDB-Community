1. Menciona tres formas en que la evidencia del perfilador o de la bitácora puede perderse sin que nadie la borre deliberadamente.

- Sobrescritura circular en system.profile: Al tratarse de una colección limitada (capped collection) con un tamaño por defecto de 1 MB, la actividad normal de la base de datos sobrescribe automáticamente los documentos más antiguos en cuanto se alcanza ese límite.
- Reinicio del servicio mongod: Vacía por completo el búfer circular en memoria de getLog y restablece el nivel de perfilado configurado en tiempo de ejecución a 0 si no fue definido de manera permanente en el archivo de configuración de arranque.
- Sobrescritura del archivo de bitácora al arrancar: Si en la configuración de mongod se utiliza systemLog.destination: file pero no se establece systemLog.logAppend: true, cada reinicio del servicio reemplaza el archivo de registro existente desde cero en lugar de anexar los nuevos eventos al final.

2. ¿Por qué el tamaño de 1 MB de system.profile es un problema de conservación y no solo de espacio?
   Porque en un servidor en producción con tráfico continuo o con el perfilador activado en nivel 2, 1 MB de registros puede llenarse en cuestión de minutos o incluso segundos. Cuando esto ocurre, el motor elimina silenciosamente los registros más antiguos para dar espacio a los nuevos sin emitir alertas ni detener el servicio, destruyendo la evidencia forense antes de que el administrador pueda revisarla o exportarla.
3. ¿Por qué getLog sirve para interpretar pero no para conservar?
   Porque el comando getLog únicamente consulta un búfer temporal en la memoria RAM del proceso mongod que retiene exclusivamente los últimos 1024 eventos ocurridos, por lo que no ofrece almacenamiento histórico a largo plazo y su contenido desaparece por completo cada vez que el servidor se reinicia o se detiene.
