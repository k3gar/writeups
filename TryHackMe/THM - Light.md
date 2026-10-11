[Light](https://tryhackme.com/room/lightroom "Light")

Es un desafío de nivel fácil diseñado para enseñar técnicas de **inyección SQL** en bases de datos **SQLite**. La aplicación simula un sistema de autenticación vulnerable que escucha en el **puerto 1337**.

- **Enfoque principal**: Los participantes deben conectarse mediante `nc` y explotar una vulnerabilidad de inyección SQL para evadir filtros de entrada.
    
- **Objetivo**: Enumerar la estructura de la base de datos, extraer credenciales de administrador (usuario: **TryHackMeAdmin**) y obtener la bandera final.

## Investiga estos conceptos si quieres intentar resolverlo antes de consultar el writeup:

- Inyección SQL
- inyección SQL basada en UNION / SQL injection UNION attacks
- Función GROUP_CONCAT en MySQL

## Ejecución:

La instrucción de la room es:

>¡Estoy trabajando en una aplicación de base de datos llamada Light! ¿Te gustaría probarlo?  
>
>Si es así, la aplicación se ejecuta en el **puerto 1337**. Puedes conectarte a él usando`nc MACHINE_IP 1337` 
>
Puedes usar el nombre de usuario`smokey` para comenzar.

Así que nos conectamos a través de Netcat. Al conectarnos, ingresamos el usuario que la plataforma nos brindó.

![[light - 1.png]]

Ya que este es un desafío de bases de datos, intentamos hacer una inyección SQL simple, ingresando el valor `'`. Al hacerlo, nos devuelve el error `Error: unrecognized token: "''' LIMIT 30"`, lo cual nos indica que la app puede ser vulnerable a inyecciones SQL, ya que el sistema no sanea correctamente la entrada del usuario, permitiendo que caracteres especiales (como comillas simples) rompan la estructura lógica de la consulta original.

![[light - 2 1.png]]

Ingresamos unos comandos de prueba, intentado recibir un error por parte del sistema, y nos damos cuenta que ciertas palabras están bloqueadas y no pueden ser ingresadas directamente.

![[light - 3.png]]

Probamos con el comando `' UniOn SeLeCt 1 '` intercambiando los valores entre mayúsculas y minúsculas. Al hacerlo, nos confirma que la query original solo selecciona 1 columna, y nos devuelve el valor que inyectamos (el 1)

![[light - 4.png]]

Luego, identificamos qué tipo/versión de base de datos estamos usando, utilizando cualquiera de las siguientes instrucciones:

```
' UniOn SeLeCt @@version '
' UniOn SeLeCt version() '
' UniOn SeLeCt sqlite_version() '
``` 

En este caso, nos damos cuenta que es una base de datos SQLite con la versión version 3.31.1

![[light - 5.png]]

Ahora, usamos la instrucción `' UniOn SeLeCt group_concat(sql) FROM sqlite_master '` en este caso, `group_concat(sql)` toma todas las filas de la columna sql y las une en una sola cadena separada por comas.

`sqlite_master` Es la "tabla del esquema": cada fila es un objeto (tabla, índice, vista) y la columna `sql` contiene su sentencia `CREATE`

![[light - 6.png]]

Esto nos permite saber que hay dos tablas disponibles junto a sus atributos. En este caso, tenemos las tablas `usertable` y `admintable` 

Ahora, utilizamos las instrucciones:

```
' UniOn SeLeCt group_concat(username) FROM usertable '

' UniOn SeLeCt group_concat(username) FROM admintable '

' UniOn SeLeCt group_concat(password) FROM admintable '
```

Para consultar los nombres de usuarios de las tablas usertable y admintable, así como la contraseña de los usuarios de la tabla admintable

![[light - 7.png]]