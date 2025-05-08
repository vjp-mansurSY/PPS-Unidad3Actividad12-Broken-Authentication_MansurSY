# PPS-Unidad3Actividad12-Broken-Authentication_MansurSY

Explotación y Mitigación de Broken Authenticatión().
Tenemos como objetivo:

> - Ver cómo se pueden hacer ataques autenticación.
>
> - Analizar el código de la aplicación que permite ataques de autenticación débil.
>
> - Implementar diferentes modificaciones del codigo para aplicar mitigaciones o soluciones.

## ¿Qué es la Autenticación débil?
---

Algunos sitios web ofrecen un proceso de registro de usuarios que automatiza (o semiautoma) el aprovisionamiento del acceso del sistema a los usuarios. Los requisitos de identidad para el acceso varían de una identificación positiva a ninguna, dependiendo de los requisitos de seguridad del sistema. Muchas aplicaciones públicas automatizan completamente el proceso de registro y aprovisionamiento porque el tamaño de la base de usuarios hace que sea imposible administrar manualmente. Sin embargo, muchas aplicaciones corporativas aprovisionarán a los usuarios manualmente, por lo que este caso de prueba puede no aplicarse.

Esto puede incluir credenciales débiles, almacenamiento inseguro de contraseñas, gestión inadecuada de sesiones y falta de protección contra ataques de fuerza bruta.

**Consecuencias de Autenticación débil:**
- Descubrimiento de credenciales de usuario.
- Ejecución de ataques de suplantación de usuarios. 

 
## ACTIVIDADES A REALIZAR
---
> Lee detenidamente la sección de vulnerabilidades de subida de archivos.  de la página de PortWigger <https://portswigger.net/web-security/authentication>
>
> Lee el siguiente [documento sobre Explotación y Mitigación de ataques de Remote Code Execution](./files/ExplotacionYMitigacionBrokenAuthentication.pdf)
> 
> También y como marco de referencia, tienes [ la sección de correspondiente de los Procesos de Registros de Usuarios del  **Proyecto Web Security Testing Guide** (WSTG) del proyecto **OWASP**.](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/03-Identity_Management_Testing/02-Test_User_Registration_Process)>


Vamos realizando operaciones:

## Operaciones previas
---
Antes de comenzar tenemos que realizar varias operaciones previas:

- Iniciar el entorno de pruebas

- Comprobar la base de datos con la que vamos a trabajar:
	- Para esta actividad tenemos una base de datos con nombre usuarios, con campos id, usuario, contrasenya.

- Descargar el diccionario de contraseñas con el que vamos a realizar un ataque de fuerza bruta.



### Iniciar entorno de pruebas

-Situáte en la carpeta de del entorno de pruebas de nuestro servidor LAMP e inicia el esce>

~~~
docker-compose up -d
~~~

 
### Creación de la Base de Datos
---

Para realizar esta actividad necesitamos acceder a una Base de datos con usuarios y contraseñas. Si ya la has creado en la actividad de Explotación y mitigación de ataques de inyección SQL, no es necesario que la crees de nuevo. Si no la has creado, puedes verlo en <https://github.com/jmmedinac03vjp/PPS-Unidad3Actividad4-InyeccionSQL> en la sección de Creación de Base de datos.

Crea la tabla de usuarios. Debería de mostrarte algó así al acceder a:

~~~
http://localhost:8080
~~~

![](images/ba1.png)

![image](https://github.com/user-attachments/assets/f0ce210d-622f-41a9-b8c7-6e4358ffe6a0)



### Instalar **hydra** en tu equipos.

Vamos a realizar un ataque de fuerza bruta para intentar recuperar las contraseñas. Esto lo haremos con el malware **hydra**

Si tu equipo es Linux, puedes instalarlo con:

~~~
sudo apt install hydra
~~~

Si tienes Windows puedes descargarlo desde la página del desarrollador: <https://www.incibe.es/servicio-antibotnet/info/Hydra>


### Descargar el diccionario de contraseñas

Podemos encontrar muchos archivos de contraseñas. Vamos a utilizar el que se encuentra en la siguiente dirección:
 <https://weakpass.com/download/90/rockyou.txt.gz>

Lo descargarmos dentro de **nuestro equipo, con el que vamos a simular serr nosotros un atacante**,y una vez descargado, lo colocamos en el directorio que deseemos, descargamos con wget y descomprimimos el archivo. En el caso de que utilizemos Linux:

~~~
cd /usr/share
wget https://weakpass.com/download/90/rockyou.txt.gz
gunzip rockyou.txt.gz
~~~

![image](https://github.com/user-attachments/assets/5d29ed71-7098-44f6-ba29-cd8a8c52c61c)


## Código vulnerable
---

El código contiene varias vulnerabilidades que pueden ser explotadas para realizar ataques de autenticación rota.

Crear al archivo **login_weak.php** con el siguiente contenido (tencuidado de sustituír **mi_password** por la contraseña de root de tu BBDD:

~~~
<?php
// creamos la conexión 
$conn = new mysqli("database", "root", "MyPassword", "SQLi");

if ($conn->connect_error) {
        // Excepción si nos da error de conexión
        die("Error de conexión: " . $conn->connect_error);
}
if ($_SERVER["REQUEST_METHOD"] == "POST" || $_SERVER["REQUEST_METHOD"] == "GET") {
        // Recogemos los datos pasados
        $username = $_REQUEST["username"];
        $password = $_REQUEST["password"];

        print("Usuario: " . $username . "<br>");
        print("Contraseña: " . $password . "<br>");

        // preparamos la consulta
        $query = "SELECT * FROM usuarios WHERE usuario = '$username' AND contrasenya = '$password'";
        print("Consulta SQL: " . $query . "<br>");

        //realizamos la consulta y recogemos los resultados
        $result = $conn->query($query);
        if ($result->num_rows > 0) {
        echo "Inicio de sesión exitoso";
        } else {
                echo "Usuario o contraseña incorrectos";
        }
}
$conn->close();

?>
<form method="post">
        <input type="text" name="username" placeholder="Usuario">
        <input type="password" name="password" placeholder="Contrasenya">
        <button type="submit">Iniciar Sesión</button>
</form>
~~~

![image](https://github.com/user-attachments/assets/4ce0cbce-6532-4ccb-b2bb-621b29f1770f)



Antes de acceder la página web, asegurarse de que el servicio está en ejecución, y si es necesario, arrancar o reiniciar el servicio.

Acceder a la pagina web aunque también podemos poner directamente el usuario y contraseña. Un ejemplo es  el siguiente enlace:

~~~
http://localhost/login_weak.php?username=admin&password=123456
~~~



Vemos que si los datos son incorrectos nos muestra que no lo es:

![](images/ba2.png)

![image](https://github.com/user-attachments/assets/2836293c-02d9-4450-b902-abdf2432f9fe)


Y si es correcta nos lo indica:

![](images/ba3.png)

![image](https://github.com/user-attachments/assets/7842b67a-a5fe-4399-8896-23d69bccce0e)


**Vulnerabilidades del código:**
1. Inyección SQL: La consulta SQL usa variables sin validación, lo que permite ataques de inyección.

2. Uso de contraseñas en texto plano: No se usa hashing para almacenar las contraseñas, lo que facilita su robo en caso de acceso a la base de datos.

3. Falta de control de intentos de inicio de sesión: No hay mecanismos de protección contra ataques de fuerza bruta.

4. Falta de gestión segura de sesiones: No se generan tokens de sesión seguros tras un inicio de sesión exitoso.


## Explotación de vulnerabilidades de Autenticación Débil

Si el usuario root de MySQL no tiene una contraseña asignada, estableced una para evitar posibles inconvenientes al trabajar con MySQL.


### Ataque de fuerza bruta con Hydra

Si el sistema no tiene un límite de intentos fallidos, se puede usar Hydra para adivinar contraseñas:

Hydra es un malware de tipo troyano bancario que se enfoca en infectar dispositivos Android para robar credenciales bancarias. Además, proporciona una puerta trasera a los atacantes que permite incluir el dispositivo como parte de una botnet y realizar otras actividades maliciosas.

En esta ocasión vamos a simular ser los atacantes y vamos a hacer un ataque de fuerza bruta con Hydra. Intentaremos acceder con todos los usuarios y las contraseñas incluidas en el diccionario rockyou.txt que hemos descargado anteriormente. 

Recordamos que seremos nosotros los atacantes, por eso desde nuestro equipo anfitrión, donde hemos descargado hydra y el diccionario, ejecutamos:

~~~
hydra -l admin -P /usr/share/rockyou.txt localhost http-post-form "/login_weak.php:username=^USER^&password=^PASS^:Usuario o contraseña incorrectos" -V
~~~



Explicación de los parámetros:

• -l el usuario con el que vamos a probar el login. 

• http-post-form: Indica que estás atacando un formulario de autenticación con método POST.

• "/login_weak.php:username=^USER^&password=^PASS^:Fallo":

	- /login_weak.php → Ruta de la página de inicio de sesión.

	- username=^USER^&password=^PASS^ → Parámetros que se envían en la solicitud POST. Hydra reemplazará ^USER^ y ^PASS^ con los valores de la lista de usuarios y contraseñas.

	- Fallo → Texto que aparece en la respuesta cuando el inicio de sesión falla. Se puede cambiar por el mensaje real de error que muestra la página cuando una contraseña es incorrecta (por ejemplo, "Usuario o contraseña incorrectos").
---

Aquí podemos ver cómo lanzamos el comando:

![](images/ba4.png)

![image](https://github.com/user-attachments/assets/47c4d0cd-e174-42a0-a560-8a1d12a2685c)


Si encontramos un resultado correcto de autenticación, vemos como nos lo muestra:

![](images/ba5.png)

![image](https://github.com/user-attachments/assets/6de1ed39-3a4e-47fc-b16c-768f9d6be8d6)


## Explotación de SQL Injection
---

Cómo ya vimos en la actividad de Inyección de SQL, el atacante puede intentar un payload malicioso en el campo de contraseña:

~~~
username: admin
password: ' OR '1'='1
~~~

Esto convertiría la consulta en:

~~~
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1';
~~~

Debido a que '1'='1' es siempre verdadero, el atacante obtendría acceso.

![](images/ba6.png)


![image](https://github.com/user-attachments/assets/d6a27dd0-d794-4380-9514-2a1a3e45bc4a)


## Mitigación: Código Seguro en PHP
---

### **Uso de contraseñas cifradas con password_hash**
---

La primera aproximación es no guardar las contraseñas en texto, sino aplicarle encriptación o hash que lo hemos visto ya en los contenidos teóricos.

Para almacenar las contraseñas hasheadas, deberemos de modificar la tabla donde guardamos los usuarios, por lo que tenemos que realizar varias operaciones:

> **Modificamos la tabla de contraseñas de la BBDD**
>
> Ejecutamos la consulta sobre la BBDD 
>
> Recuerda que:
>
> - Accedemos al contenedor de la BBDD:
>
~~~
 docker exec -it lamp-mysql8 /bin/bash
~~~


>
> - Nos conectamos a la Base de Datos como usuario root con mysql y despues ejecutar la consulta).
>
~~~
 mysql -u root -p
~~~
>
> - Y seleccionamos la BBDD y modificamos la tabla:
>
~~~
 USE SQLi
 ALTER TABLE usuarios MODIFY contrasenya VARCHAR(255) NOT NULL; 
~~~
>
![](images/ba7.png)

![image](https://github.com/user-attachments/assets/9376b78d-619e-484d-b26a-3649e7d0f8ac)



>Creamos la función **ạdd_user.php** para introducir los usuarios con su contraseña hasheada (Acuérdate de cambiar MiContraseña por la tuya de root):

~~~
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

// Conexión
$conn = new mysqli("database", "root", "MiContraseña", "SQLi"); 
// ← Usa "localhost" si no estás en Docker
if ($conn->connect_error) {
    die("Conexión fallida: " . $conn->connect_error);
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Verificamos campos
    if (isset($_POST["username"]) && isset($_POST["password"])) {
        $username = $_POST["username"];
        $password = $_POST["password"];

        // Hasheamos contraseña
        $hashed_password = password_hash($password, PASSWORD_DEFAULT);

        // Insertamos usuario
        $stmt = $conn->prepare("INSERT INTO usuarios (usuario, contrasenya) VALUES (?, ?)");
        if ($stmt === false) {
            die("Error en prepare: " . $conn->error);
        }

        $stmt->bind_param("ss", $username, $hashed_password);

        if ($stmt->execute()) {
            echo "✅ Usuario insertado correctamente.";
        } else {
            echo "❌ Error al insertar usuario: " . $stmt->error;
        }

        $stmt->close();
    } else {
        echo "⚠️ Por favor, rellena todos los campos.";
    }
}

$conn->close();
?>

<form method="post">
    <input type="text" name="username" placeholder="Usuario" required>
    <input type="password" name="password" placeholder="Contrasenya" required>
    <button type="submit">Crear Usuario</button>
</form>
~~~

![image](https://github.com/user-attachments/assets/b3e6b13a-72d6-4b3c-8850-200a3b29174c)


En la función **pasword_hash()"** utilizamos la función por defecto: **PASSWORD_DEFAULT** que usa actualmente **BCRYPT**, pero se actualizará automáticamente en versiones futuras de PHP. Si deseas más control, puedes usar **PASSWORD_BCRYPT** o **PASSWORD_ARGON2ID**.

>Como vemos, una vez ejecutado nos informa que el usuario raul con contraseña 123456 ha sido insertado.
>
>![](images/ba8.png)

![image](https://github.com/user-attachments/assets/001580e2-91a1-498b-8e0a-2d52bc996bba)


 Lo podemos ver accediendo al servicio phpmyadmin: `http://localhost:8080`

![](images/ba9.png)

![image](https://github.com/user-attachments/assets/26cd24bb-ee29-481a-a625-f976a41b3481)


 También puedes obtener los usuarios conectandote a la base de datos y ejecutando la consulta:

 ~~~
SELECT * from usuarios
~~~



La función **password_hash()** con **PASSWORD_BCRYPT** genera un hash de hasta 60 caracteres, y con
PASSWORD_ARGON2ID, incluso más (hasta 255). Por eso, se necesita que la columna pueda almacenarlos
adecuadamente.

Aplicando mitigaciones de uso de contraseñas con password_hash tendríamos el siguiente archivo: **login_weak1.php**:
(Recuerda que tienes que cambiar miContraseña por tu contraseña de root)
~~~
<?php
// creamos la conexión 
$conn = new mysqli("database", "root", "MyPassword", "SQLi");

if ($conn->connect_error) {
        // Excepción si nos da error de conexión
        die("Error de conexión: " . $conn->connect_error);
}
if ($_SERVER["REQUEST_METHOD"] == "POST" || $_SERVER["REQUEST_METHOD"] == "GET") {
        // Recogemos los datos pasados
        $username = $_REQUEST["username"];
        $password = $_REQUEST["password"];

        print("Usuario: " . $username . "<br>");
        print("Contraseña: " . $password . "<br>");

        // NO PREVENIMOS SQL INJECTION, SOLO SE AGREGA PASSWORD_HASH
        $query = "SELECT contrasenya FROM usuarios WHERE usuario = '$username'";
        print("Consulta SQL: " . $query . "<br>");

        //realizamos la consulta y recogemos los resultados
        $result = $conn->query($query);
        if ($result->num_rows > 0) {
                $row = $result->fetch_assoc();
                $hashed_password = $row["contrasenya"];
                // Verificación de contraseña hasheada
                if (password_verify($password, $hashed_password)) {
                        echo "Inicio de sesión exitoso";
                } else {
                        echo "Usuario o contraseña incorrectos";
                }
        } else {
                echo "Usuario no encontrado";
        }
}
$conn->close();

?>
<form method="post">
        <input type="text" name="username" placeholder="Usuario">
        <input type="password" name="password" placeholder="Contrasenya">
        <button type="submit">Iniciar Sesión</button>
</form>
~~~

![image](https://github.com/user-attachments/assets/5a64b78b-4012-47c0-8697-d0e43fb6699d)


Como vemos en la siguiente imagen nos da un login exitoso:

![](images/ba10.png)

![image](https://github.com/user-attachments/assets/043a233c-553e-48f2-a021-022d8bfbb30f)


También puedes probar a usuarlos introduciendo en el navegador:

~~~
http://localhost/login_weak1.php?username=raul&password=123456
~~~

![image](https://github.com/user-attachments/assets/d9c52969-3685-4737-88bb-9abfce9560a9)


Si introducimos datos no correcto dará el mensaje de "Usuario o contraseña no correctos"

~~~
http://localhost/login_weak1.php?username=raul&password=1234
~~~

![](images/ba10.png)

![image](https://github.com/user-attachments/assets/7f657ef2-3a47-4f5f-9d25-dba3b269d584)


### Uso de consultas preparadas

La siguiente aproximación es usar consultas preparadas, así evitamos ataques de SQL injection.

Creamos el archivo **login_weak2.php** con el siguiente contenido:

~~~
<?php
// Conexión
$conn = new mysqli("database", "root", "MyPassword", "SQLi");
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// Procesamos petición POST o GET
if ($_SERVER["REQUEST_METHOD"] == "POST" || $_SERVER["REQUEST_METHOD"] == "GET") {
    $username = $_REQUEST["username"];
    $password = $_REQUEST["password"];

    print("Usuario: " . $username . "<br>");
    print("Contraseña: " . $password . "<br>");

    // Consulta segura con prepare + bind
    $query = "SELECT contrasenya FROM usuarios WHERE usuario = ?";
    $stmt = $conn->prepare($query);
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $stmt->store_result();

    print("Consulta SQL (preparada): " . $query . "<br>");

    if ($stmt->num_rows > 0) {
        $stmt->bind_result($hashed_password);
        $stmt->fetch();

        // Comprobamos si la contraseña ingresada coincide con el hash
        if (password_verify($password, $hashed_password)) {
            echo "✅ Inicio de sesión exitoso";
        } else {
            echo "❌ Usuario o contraseña incorrectos";
        }
    } else {
        echo "❌ Usuario no encontrado";
    }

    $stmt->close();
}
$conn->close();
?>

<!-- Formulario -->
<form method="post">
    <input type="text" name="username" placeholder="Usuario">
    <input type="password" name="password" placeholder="Contrasenya">
    <button type="submit">Iniciar Sesión</button>
</form>

~~~

![image](https://github.com/user-attachments/assets/57619ddb-211e-4bf2-b257-f183b43b18f6)


Como vemos, hemos usado consutas paremetrizadas y además hemos utilizado las funciones para manejar las contraseñas hasheadas:

>🔐 ¿Cómo funciona?
>
>password_hash($password, PASSWORD_DEFAULT) genera una contraseña hasheada segura.0
>
>password_verify($input, $hash_guardado) verifica si la contraseña ingresada coincide con la almacenada.>


### * Implementar bloqueo de cuenta tras varios intentos fallidos
Para bloquear la cuenta después de 3 intentos fallidos, podemos hacer lo siguiente:
1. Añadir un campo failed_attempts en la base de datos para contar los intentos fallidos. 

2. Registrar el timestamp del último intento fallido con un campo last_attempt para poder restablecer los intentos después de un tiempo.

3. Modificar la lógica del login:

	- Si el usuario tiene 3 intentos fallidos, bloquear la cuenta.
	
	- Si han pasado, por ejemplo, 15 minutos desde el último intento, restablecer los intentos fallidos.

	- Si el login es exitoso, reiniciar los intentos fallidos a 0.

**Modificación en la Base de Datos**

Accede a la BBDD como hemos hecho al principio de la actividad y modificala de la siguiente forma: 

~~~
USE SQLi
ALTER TABLE usuarios ADD failed_attempts INT DEFAULT 0;
ALTER TABLE usuarios ADD last_attempt TIMESTAMP NULL DEFAULT NULL;
~~~

![image](https://github.com/user-attachments/assets/df2d223a-415b-422b-8156-9751aba2ac2f)


Vemos como se han añadido las columnas indicadas:

![](images/ba1.png)

![image](https://github.com/user-attachments/assets/e22a4fb9-20b5-4403-85f1-e183698491db)


**Código seguro**

Crea el ficher **login_weak3.php** con el siguiete contenido (recuerda cambiar la contraseña):

~~~
<?php
// Conexión
$conn = new mysqli("database", "root", "MyPassword", "SQLi");
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// Procesamos petición
if ($_SERVER["REQUEST_METHOD"] == "POST" || $_SERVER["REQUEST_METHOD"] == "GET") {
    $username = $_REQUEST["username"];
    $password = $_REQUEST["password"];

    print("Usuario: " . $username . "<br>");
    print("Contraseña: " . $password . "<br>");

    // Obtenemos datos del usuario
    $query = "SELECT contrasenya, failed_attempts, last_attempt FROM usuarios WHERE usuario = ?";
    $stmt = $conn->prepare($query);
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $stmt->store_result();

    if ($stmt->num_rows > 0) {
        $stmt->bind_result($hashed_password, $failed_attempts, $last_attempt);
        $stmt->fetch();

        $current_time = new DateTime();
        $is_blocked = false;

        // Si la cuenta está bloqueada (3 intentos fallidos)
        if ($failed_attempts >= 3 && $last_attempt !== null) {
            $last_attempt_time = new DateTime($last_attempt);
            $interval = $current_time->getTimestamp() - $last_attempt_time->getTimestamp();

            if ($interval < 900) { // Menos de 15 minutos
                $remaining = 900 - $interval;
                $minutes = floor($remaining / 60);
                $seconds = $remaining % 60;
                echo "⛔ Cuenta bloqueada. Intenta nuevamente en {$minutes} minutos y {$seconds} segundos.";
                $is_blocked = true;
            }
        }

        if (!$is_blocked) {
            // Verificamos contraseña
            if (password_verify($password, $hashed_password)) {
                echo "✅ Inicio de sesión exitoso";

                // Reiniciar intentos fallidos
                $reset_query = "UPDATE usuarios SET failed_attempts = 0, last_attempt = NULL WHERE usuario = ?";
                $reset_stmt = $conn->prepare($reset_query);
                $reset_stmt->bind_param("s", $username);
                $reset_stmt->execute();
                $reset_stmt->close();
            } else {
                // Incrementar intentos
                $failed_attempts++;
                echo "❌ Usuario o contraseña incorrectos (Intento $failed_attempts de 3)";

                $update_query = "UPDATE usuarios SET failed_attempts = ?, last_attempt = NOW() WHERE usuario = ?";
                $update_stmt = $conn->prepare($update_query);
                $update_stmt->bind_param("is", $failed_attempts, $username);
                $update_stmt->execute();
                $update_stmt->close();
            }
        }
    } else {
        echo "❌ Usuario no encontrado";
    }

    $stmt->close();
}
$conn->close();
?>

<!-- Formulario -->
<form method="post">
    <input type="text" name="username" placeholder="Usuario">
    <input type="password" name="password" placeholder="Contrasenya">
    <button type="submit">Iniciar Sesión</button>
</form>
~~~

![image](https://github.com/user-attachments/assets/aadfefd1-e4f3-42f8-8ccb-d1724e1d8e22)


🔍 Qué hace este código:

- Si el usuario tiene 3 fallos y han pasado menos de 15 minutos, la cuenta se bloquea temporalmente.

- Si han pasado más de 15 minutos, los intentos se reinician automáticamente.

- Si el login es exitoso, se ponen los intentos a cero y se borra el last_attempt.

### Implementar autenticación multifactor (MFA)

Para añadir MFA (Autenticación Multifactor) al sistema de login, seguiremos estos pasos:

> Pasos para Implementar MFA
> 1. Generar un código de verificación temporal (OTP) de 6 dígitos.
>
> 2. Enviar el código OTP al usuario mediante correo electrónico o SMS (en este caso, usaremos correo simulado con una archivo PHP.
>
> 3. Crear un formulario para que el usuario ingrese el código OTP después de iniciar sesión.
>
> 4. Verificar el código OTP antes de permitir el acceso.
>
🧩 ¿Qué vamos a crear?

- Modificaciones en la base de datos:

	- Campos mfa_code (VARCHAR) y mfa_expires (DATETIME).

- Flujo dividido en dos archivos:

	- login_weak4.php: usuario y contraseña → si correctos, se genera el MFA.


	- verificar_mfa.php: el usuario introduce el código que se le muestra.

	- mostrar_codigo.php: archivo que muestra el código generado.

**1. Modificación en la Base de Datos**

Accede a la BBDD como hemos hecho al principio de la actividad y modificala de la siguiente forma: 

~~~
USE SQLi
ALTER TABLE usuarios ADD failed_attempts INT DEFAULT 0;
ALTER TABLE usuarios ADD last_attempt TIMESTAMP NULL DEFAULT NULL;
~~~

![image](https://github.com/user-attachments/assets/ac4574d4-8b3a-4035-b1fd-5d95cf077a46)


**🔐 2. login_weak4.php (login + generación del código)**

Crea el archivo login_weak4.php con el siguiente contenido (recuerda cambiar la contraseña):

~~~
<?php
$conn = new mysqli("database", "root", "MyPassword", "SQLi");
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

session_start();

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $username = $_POST["username"];
    $password = $_POST["password"];

    $query = "SELECT contrasenya FROM usuarios WHERE usuario = ?";
    $stmt = $conn->prepare($query);
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $stmt->store_result();

    if ($stmt->num_rows > 0) {
        $stmt->bind_result($hashed_password);
        $stmt->fetch();

        if (password_verify($password, $hashed_password)) {
            // ✅ Login correcto - generar MFA
            $mfa_code = strval(rand(100000, 999999));
            $expires = (new DateTime('+5 minutes'))->format('Y-m-d H:i:s');

            // Guardar código MFA
            $update = $conn->prepare("UPDATE usuarios SET mfa_code = ?, mfa_expires = ? WHERE usuario = ?");
            $update->bind_param("sss", $mfa_code, $expires, $username);
            $update->execute();

            // Guardar usuario en sesión para MFA
            $_SESSION["mfa_user"] = $username;

            // Redirigir a mostrar el código y luego a verificación
            header("Location: mostrar_codigo.php?code=$mfa_code");
            exit();
        } else {
            echo "❌ Contraseña incorrecta.";
        }
    } else {
        echo "❌ Usuario no encontrado.";
    }
    $stmt->close();
}
$conn->close();
?>

<form method="post">
    <input type="text" name="username" placeholder="Usuario" required>
    <input type="password" name="password" placeholder="Contraseña" required>
    <button type="submit">Iniciar sesión</button>
</form>

~~~

![image](https://github.com/user-attachments/assets/a012b90f-9498-4629-bf2b-299f326cf09c)





**🪪 3. mostrar_codigo.php**


Creamos el archivo **mostrar_codigo.php** con el que visualizaremos el código enviado. Esto simula el ver el código en el email. 

~~~
<?php
$code = $_GET["code"] ?? "XXXXXX";
echo "<h2>🔐 Tu código MFA es: <strong>$code</strong></h2>";
echo "<a href='verificar_mfa.php'>Ir a verificación MFA</a>";
?>
~~~

![image](https://github.com/user-attachments/assets/5feb777f-d321-4305-ac74-cfd79c48046d)

![image](https://github.com/user-attachments/assets/4ee6bf00-a364-4b3f-abf5-8d0f67846191)


**✅ 4. verificar_mfa.php (verificación del código)**

Creamos el archivo **verificar_mfa.php** que nos indicará si el código introducido es correcto (recuerda cambiar la contraseña).

~~~
<?php
session_start();
$conn = new mysqli("database", "root", "MyPassword", "SQLi");
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

if (!isset($_SESSION["mfa_user"])) {
    die("⚠️ No hay sesión activa para MFA.");
}

$username = $_SESSION["mfa_user"];

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $code_input = $_POST["mfa_code"];

    $query = "SELECT mfa_code, mfa_expires FROM usuarios WHERE usuario = ?";
    $stmt = $conn->prepare($query);
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $stmt->bind_result($mfa_code, $mfa_expires);
    $stmt->fetch();

    $now = new DateTime();
    $expires_time = new DateTime($mfa_expires);

    if ($code_input === $mfa_code && $now < $expires_time) {
        echo "✅ Autenticación multifactor exitosa. Bienvenido, $username.";

        // Limpieza del código MFA
        $clear = $conn->prepare("UPDATE usuarios SET mfa_code = NULL, mfa_expires = NULL WHERE usuario = ?");
        $clear->bind_param("s", $username);
        $clear->execute();

        session_destroy(); // o puedes mantener sesión como autenticado
    } else {
        echo "❌ Código incorrecto o expirado.";
    }
    $stmt->close();
}
$conn->close();
?>

<form method="post">
    <input type="text" name="mfa_code" placeholder="Código MFA" required>
    <button type="submit">Verificar Código</button>
</form>

~~~

![image](https://github.com/user-attachments/assets/84a9dcac-2571-44ee-bc0f-19668394f558)



🧪 Flujo de prueba

- En login.php, introduces usuario y contraseña.

- Si están bien, se genera un código y vas a mostrar_codigo.php.

![](images/ba13.png)

- Desde ahí, clicas a verificar_mfa.php e introduces el código.

![](images/ba14.png)

![image](https://github.com/user-attachments/assets/04b938cf-082c-49a3-8523-58f6beeb372b)


![image](https://github.com/user-attachments/assets/d4967658-b8af-45f2-805d-d280ce5de014)



🔒 Flujo completo del Login con MFA

1. Usuario ingresa su usuario y contraseña.

2. Si las credenciales son correctas, se genera un código OTP y se guarda en la BD.

3. Se envía el código OTP al usuario por correo electrónico (fichero emails_simulados.txt).

4. Usuario ingresa el código OTP en un formulario.

5. El sistema verifica si el código es válido y no ha expirado.

6. Si es correcto, el usuario accede; si no, se muestra un error.


🚀 Beneficios de este Sistema MFA

✔  Mayor seguridad contra accesos no autorizados.

✔  Protege contra ataques de fuerza bruta, incluso si la contraseña es robada.

✔  Fácil de extender a SMS o aplicaciones como Google Authenticator.


## ENTREGA

> __Realiza las operaciones indicadas__

> __Crea un repositorio  con nombre PPS-Unidad3Actividad12-Tu-Nombre donde documentes la realización de ellos.__

> No te olvides de documentarlo convenientemente con explicaciones, capturas de pantalla, etc.

> __Sube a la plataforma, tanto el repositorio comprimido como la dirección https a tu repositorio de Github.__
