Pagina para el examen de la unidad 3 
---
El primer paso para comenzar con el proyecto es que todos los integrantes del equipo se unan a una misma sesión en conjunto de Visual Studio Code para editar el mismo proyecto. Esto se logra gracias a la extensión de VSC "Live Share", esta extensión se instala en el apartado de "descargar extensiones". Ademas en necesario descargar más extensiones como: 

```
npm install bcrypt
```
y 
```
npm install -g nodemon
```

<img width="509" height="454" alt="image" src="https://github.com/user-attachments/assets/4f34cbbd-8120-4c95-8e91-573bcd5d8752" />

Una vez instalada debe decidirse quien sera el host del server y este usuario debera compartir el link de la sesión en conjunto mediante el menú que aparece en la parte inferior de la ventana.

<img width="1100" height="399" alt="image" src="https://github.com/user-attachments/assets/f3c6ab9c-bc6c-47b1-bec2-ff00d38bc01e" />

El siguiente paso es crear un archivo ".env" en el nucleo del proyecto, este archivo permite guardar variables y llamarlas desde otros archivos.
<img width="466" height="304" alt="image" src="https://github.com/user-attachments/assets/ae11f625-9120-42da-9530-45fc970a7e58" />

El siguiente paso es crear un archivo llamado "nodemon.json" y agregamos el siguiente bloque de código 
```
{
  "watch": ["server.js", "public"],
  "ext": "js,html,css,json",
  "ignore": ["node_modules", ".git"],
  "exec": "node server.js"
}
```
Este código permite inicios automaticos del host, esto evita que se tenga reiniciar el servidor manualmente con cada cambio que se realiza en el archvio "server.js". 
En la parte de scripts del archivo "package.json" cambiamos en la parte de start, cambiamos:
```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon",
    "dev": "nodemon server.js"
  },
```

Para crear el login creamos un documento html con el siguiente codigo:
```
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Iniciar Sesión</title>
   <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h2>Iniciar Sesión</h2>
  <form action="/login" method="POST">
    <label for="nombre_usuario">Nombre de Usuario:</label>
    <input type="text" id="nombre_usuario" name="nombre_usuario" required>
    <br>
    <label for="password">Contraseña:</label>
    <input type="password" id="password" name="password" required>
    <br>
    <button type="submit">Iniciar Sesión</button>
  </form>
  <p>¿No tienes cuenta? <a href="/registro">Regístrate aquí</a></p>
</body>
</html>
```

Y en _Server_ agruegar la siguiente funcion: 
```
app.post('/login', (req, res) => {
    const { nombre_usuario, password } = req.body;
    console.log(req.body);

    const query = 'SELECT * FROM usuarios WHERE nombre = ?';
    connection.query(query, [nombre_usuario], (err, results) => {
        if (err) {
          let html =`<html>
          <head>
            <link rel="stylesheet" href="/styles.css">
            <title>Medicos</title>
          </head>
          <body>
            <p></p>
            <h4>Error de Operacion</h4>
            <p></p>
            <h3> No se pudo obtener al usuario  </h3>
            <p></p>
            <button onclick="window.location.href='/'">Regresar</button>
          </body>
          </html>
      
      
          </html>`;
          
          return res.send(html);
        }

        if (results.length === 0) {
           let html =`<html>
          <head>
            <link rel="stylesheet" href="/styles.css">
            <title>Medicos</title>
          </head>
          <body>
            <p></p>
            <h4>Error de Operacion</h4>
            <p></p>
            <h3> No se pudo obtener al usuario  </h3>
            <p></p>
            <button onclick="window.location.href='/'">Regresar</button>
          </body>
          </html>
          
          
          </html>`;
            return res.send(html);
        }

        const user = results[0];

        const isPasswordValid = bcrypt.compareSync(password, user.password_hash);
        if (!isPasswordValid) {
          let html =`<html>
          <head>
            <link rel="stylesheet" href="/styles.css">
            <title>Medicos</title>
          </head>
          <body>
            <p></p>
            <h4>Error de Operacion</h4>
            <p></p>
            <h3> Contrasena incorrecta  </h3>
            <p></p>
            <button onclick="window.location.href='/'">Regresar</button>
          </body>
          </html>
          
          
          </html>`;

            return res.send(html);
        }

        req.session.user = {
            id: user.id,
            username: user.nombre_usuario,
            tipo_usuario: user.rol
        };

        res.redirect('/');
    });
});
```
