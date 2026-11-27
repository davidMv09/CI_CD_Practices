Una tabla con una fila por problema encontrado:

| Línea | Problema | Riesgo concreto | Corrección propuesta |
|---|---|---|---|
| (número) | Qué está mal | Qué puede pasar, y a quién | Cómo se escribe bien |
|7|Permisos de write-all| Cada job podria escribir todo lo que el token le permita | Restringir los permisos a lectura y en caso de ser necesario de empaquetar.
|10| API_TOKEN expuesto| Cualquiera puede leer o escribir el valor de la variable| Agregar el valor como variable de ambiente.