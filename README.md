Pagina para el examen de la unidad 3 
## Instalar extensiones
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

## Env y Nodemon
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
## Login
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

Para que solo usuarios logeados en la pagina puedan acceder a los distintos apartados debe ponerse en server: 
```
app.get('/', requireLogin, (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```

Para poder destruir la sesión en caso de querer cerrar la sesión se coloca en server el siguiente bloque de codigo:
```
app.get('/logout', (req, res) => {
  req.session.destroy();
  res.redirect('/login');
});
```

## Roles (ADMIN/ASISTENTE/AUDITOR) aplicados en rutas	
Es necesario crearse varias tablas en la base de datos, en especial una que contenga los codigos de acceso para registrarse como tipo de usuario entre los diferentes que hay. 
```
CREATE TABLE tipo_usuario (
    codigo_acceso VARCHAR(20),
    tipo_usuario VARCHAR(10)
);
```

En server se debe agregar:
```
app.get('/tipo-usuario', requireLogin, (req, res) => {
    res.json({ tipo_usuario: req.session.user.tipo_usuario });
});
```
Para poner escoger que tipos de usuario tengas acceso a los apartados de la página.

## Búsqueda en vivo	
Para realizar la busqueda en vivo, es necesario agregar en server.js el siguiente bloque:
```
// Búsqueda en vivo de instrumentos
app.get('/api/instrumentos/buscar', requireLogin, (req, res) => {
  const q = req.query.q || '';
  const sql = `
    SELECT * FROM instrumentos
    WHERE nombre LIKE ? OR categoria LIKE ? OR estado LIKE ? OR ubicacion LIKE ?
    LIMIT 50
  `;
  const search = `%${q}%`;
  connection.query(sql, [search, search, search, search], (err, results) => {
    if (err) {
      console.error(err);
      return res.status(500).json({ error: 'Error consultando instrumentos' });
    }
    res.json(results);
  });
});
```

Y en el archivo instrumentos.html agregar el apartado visual:
```
// Búsqueda en vivo
document.getElementById("searchInput").addEventListener("input",(e)=>{
  loadInstrumentos(e.target.value);
});
```

## CRUD de instrumentos	
Para actualizar el estado de un instrumento o borrarlo, es necesario agregar el CRUD en el mismo archivo donde esta la tabla, en este caso se debe de colocar en instrumento.html:
```

// Eliminar instrumento
async function deleteInstrumento(id){
  if (!confirm("¿Confirmas eliminar este instrumento?")) return;
  const res = await fetch(`/api/instrumentos/${id}`,{method:"DELETE"});
  if(res.ok) loadInstrumentos();
}

// Cambiar estado
async function cambiarEstado(id){
  const nuevoEstado = prompt("Ingrese el nuevo estado del instrumento:");

  if (!nuevoEstado) return;

  const res = await fetch(`/api/instrumentos/${id}/estado`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ nuevo_estado: nuevoEstado })
  });

  if(res.ok){
    alert("Estado actualizado correctamente");
    loadInstrumentos();
  } else {
    alert("Error al actualizar el estado");
  }
}
```

## Excel (subir y descargar)	
Para subir a excel se agrega un apartado en index.html:
```
  <form action="/cargar_instrumento" method="POST" enctype="multipart/form-data">
    <label>Subir archivo Excel (.xlsx):</label>
    <input type="file" name="excelFile" accept=".xlsx" required>
    <button type="submit">Cargar Instrumentos</button>
  </form>
```

En server.js se debe agregar las siguientes acciones que nos permitiran subir los documentos de excel y cargar los instrumentos en la tabla, y también descargar la tabla actual.
```
app.get('/descarga_instrumentos', (req, res) => {
  const sql = `SELECT * FROM instrumentos`;
  connection.query(sql, (err, results) => {
    if (err) throw err;

    const worksheet = xlsx.utils.json_to_sheet(results);
    const workbook = xlsx.utils.book_new();
    xlsx.utils.book_append_sheet(workbook, worksheet, 'instrumentos');

    const filePath = path.join(__dirname, 'uploads', 'instrumentos.xlsx');

    xlsx.writeFile(workbook, filePath);
    res.download(filePath, 'Lista Instrumentos.xlsx');
  });
});
//ruta para manejar la carga de paciente 

app.post('/cargar_instrumento', upload.single('excelFile'), (req, res) => {
  const filePath = req.file.path;
  const workbook = xlsx.readFile(filePath);
  const sheetName = workbook.SheetNames[0];
  const data = xlsx.utils.sheet_to_json(workbook.Sheets[sheetName]);

  data.forEach(row => {
    //CAMBIO SUTIL: Usar los nombres correctos de las columnas
    const { Nombre, Categoria, Estado, Ubicacion } = row;
    const sql = `INSERT INTO instrumentos (nombre, categoria, estado, ubicacion) VALUES (?, ?, ?, ?)`;
    connection.query(sql, [Nombre, Categoria, Estado, Ubicacion], err => {
      if (err) throw err;
    });
  });

  res.send('<h1>Archivo cargado y datos guardados</h1><a href="/">Volver</a>');
});
```

## Navbar
Para crear la navbar es necesario crear un archivo llamadao "navbar.html", que contenga lo siguiente:
```
<nav>
  <ul id="menu"></ul>
</nav>
<script>
  window.onload = () => {
    fetch('/menu')
      .then(res => res.json())
      .then(data => {
        const menu = document.getElementById('menu');
        data.forEach(item => {
          const li = document.createElement('li');
          li.innerHTML = `<a href="${item.url}">${item.nombre}</a>`;
          menu.appendChild(li);
        });
      });
  };
</script>
```

Y en server.js:
```
app.get('/menu', (req, res) => {
  const menuItems = [
    { nombre: 'Inicio', url: '/' },
    { nombre: 'Cerrar Sesion ', url: '/login.html' },
    { nombre: 'instrumentos', url: '/instrumento.html' }
  ];
  res.json(menuItems);
});
```

Esto permitira que las rutas de la navbar realmente manden a los distintos apartados de la pagina web, también permitirá que la barra se quede fija. 
