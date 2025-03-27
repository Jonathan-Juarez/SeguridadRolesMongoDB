# Seguridad y roles en mongoDB

## Introducción
En este laboratorio sobre la seguridad y roles en mongoDB se observa la importancia que presenta la protección de la base de datos, manteniendo su integridad con la privacidad y la accesibilidad que, en algunas ocasiones, algunos no lo consideran tan necesario. De forma predeterminada, MongoDB no tiene control de autenticación, lo que puede ser riesgoso si no se mantiene medidas de seguridad. En el laboratorio se ha habilitado la autenticación en mongoDB y se ha creado usuarios que tienen roles específicos con una autorización diferente en la base de datos para realizar diversas pruebas. Por último, se hizo el intento de configurar una conexión segura que usa SSL/TLS.

## Pasos seguidos con captura de pantalla
Los pasos seguidos fueron definidos en el documento proporcionado por el docente, facilitando el proceso. En resumen consistió en lo siguiente:
1. Se habilitó la autenticación en MongoDB editando el archivo de configuración mongod.cfg, donde se activó ```security: authorization: enabled```.
![securityEnabledCfg](/assets/securityEnabledCfg.png)
2. Se reinició los servidores de MongoDB para implementar los cambios en la base de datos. Esto se realizó con los comandos ejecutados en Powershell Administrador: ```net stop MongoDB``` y ```net start MongoDB```. 
![rebootMongoDB](/assets/rebootMongoDB.png)
3. Se creó tres usuarios y se les asignó un rol diferente a cada uno. De primera se tuvo que ingresar a la base de datos dependiendo el rol en mongosh y después crearlos:
* adminUser con permisos de administración en la base de datos admin.
![createAdminUser](/assets/createAdminUser.png)
* readUser con permisos de solo lectura en la base de datos miBaseDatos.
![createReadUser](/assets/createReadUser.png)
* WriteUser con permisos de lectura y escritura en la misma base de datos.
![createWriteUser](/assets/createWriteUser.png)
4. Se validaron los permisos de cada usuario ejecutando diferentes operaciones, tanto las que tenían autorización o no:
* Validación de adminUser: El usuario que permitió crear los demás usuarios fue un Admin, lo cual demuestra su funcionamiento. Esto se visualiza en paso anterior de creación de usuarios. Este usuario tiene acceso a todo y se activa ingresando a la DB admin.
* Validación de WriteUser: De primera se cambia al usuario writeUser. Se crea una colección, se ingresa un dato y se busca, lo cual tiene acceso. Al momento de intentar crear un nuevo usuario, marca que no tiene autorizado, indicando que cumple con su papel de solo leer y escribir.
![validationWriteUser](/assets/validationWriteUser.png)
* Validación de readUser: Se ingresa al usuario readUser y se ingresa un dato y se intenta eliminar lo ingresado con el usuario anterior, sin embargo, no permite. La única acción que tiene acceso es a la lectura (find).
![validationReadUser](/assets/validationReadUser.png)

5. Configuración de conexión segura con SSL/TLS:
* Se genera un certificado SSL con OpenSSL con ```openssl req -newkey rsa:2048 -new -x509 -days 365 -nodes -out C:\mongodb-cert.pem -keyout C:\mongodb-key.pem```, el cual nos solicita ingresar nuestros datos para el certificado. 
![certificateGeneration](/assets/certificateGeneration.png)
* Teniendo esto, se combinan las claves en un solo archivo con ```type C:\mongodb-key.pem C:\mongodb-cert.pem > C:\mongodb.pem```, lo cual genera un nuevo archivo mongodb.pem que posee el contenido de las dos claves.
![keyCombination](/assets/keyCombination.png)
6. Nuevamente se modifica el archivo mongod.cfg:
* Se agrega ```ssl: mode: requireSSL PEMKeyFile: C:\mongodb.pem```.
![UseSSLCfg](/assets/UseSSLCfg.png)
* Se reinicia los servidores de mongoDB, pero no permitió el reinicio, devolviendo que el servicio no está respondiendo a la función de control. Para continuar con los demás pasos, se omite el paso 6.
![rebootMongoDBFailed](/assets/rebootMongoDBFailed.png)
7. Por último, se realizó una conexión a MongoDB desde Compass, donde se validaron cada de los usuarios:
* Para conectarse a adminUser se tiene que ingresar el nombre del usuario, contraseña y la base de datos de la que tendrá control.
![AuthenticationAdminCompass](/assets/AuthenticationAdminCompass.png)
 Este usuario dio acceso a todas las bases de datos del localhost.
![connectAdminUserCompass](/assets/connectAdminUserCompass.png)
* La conexión con readUser resultó que solo podía ver la base de datos miBaseDatos, al igual que  no permitía modificar ni eliminar nada, los buscar.
![connectReadUserCompass](/assets/connectReadUserCompass.png)
![findReadUserCompass](/assets/findReadUserCompass.png)
![deleteReadUserCompass](/assets/deleteReadUserCompass.png)
* Al conectar con writeUser se observó solo mostraba la base de datos asignada, como el readUser, sin embargo, permitió insertar un dato, lo cual se ve refleado en la segunda captura.
![insertWriteUserCompass](/assets/insertWriteUserCompass.png)
![connectWriteUserCompass](/assets/connectWriteUserCompass.png)

## Problemas encontrados y soluciones aplicadas
Al momento de realizar el laboratorio se presentaron dos errores, uno bastante pequeño y otro que no se encontró solución:
* Error al intentar conectar con SSL: No fue posible encender MongoDB después de habilitar SSL en el archivo mongod.cfg. Una solución que se intentó fue al notar las versiones, ya que en la antigua versión usa SSL ```ssl: mode: requireSSL PEMKeyFile: C\:mongodb.pem```, pero la nueva versión usa TLS ```tls: mode: requireTLS certificateKeyFile: C:\mongodb.pem```. Sin embargo, apesar de este cambio, no se presentó un resultado positivo en la activación, por lo que se descartó esta configuración para continuar con el laboratorio.

* Error de autenticación en MongoDB Compass: Al colocar admin en Authentication Database, marcaba un error, por lo que lo quité y la conexión fue exitosa. Este campo realmente es necesario, pero solo me fue funcional con los otros usuarios.

## Conclusión sobre la importancia de la seguridad en bases de datos. 
Después de culminar el laboratorio se refuerza la idea de que hacer configuraciones sólidas para seguridad es fundamental para que todo quede protegido y se elimnen los riesgos de que ser robado sin permiso (hackeo). Al autenticar usuarios y autorizar roles específicos, se puede tener un control mucho mejor sobre acciones a realizar según sea la necesidad. Los usuarios y roles van más enfocados en el uso empresarial, asignado cada usuario basado al rango de los empleados, ya que no sirve de nada tener varios usuarios para un mismo individuo.  Implementar buenas prácticas de seguridad cuando se trabaja con MongoDB es fundamental para proteger los datos de uno mismo e incluso el de los clientes. 