# **APP2**
<div align="justify">

**Objetivo de la APP2:** 

en la app2 se añade todo lo que contiene la app1, es decir que sus servicios son tecnicamente iguales, tiene un CRUD, muestra lo que tendria que tener como datos la BD, esto es para el balanceador, como se menciona en la configuracion del balanceador esta app2 se muestra en caso de que falle la app1, basicamente funciona como su espejo y esta conectada a la misma BD

## **Configuración de IP estatica**

* **Cambio de IP dinámica a estatica:**
```bash
 sudo nano /etc/netplan/50-cloud-init.yaml
   ```
Dentro del archivo /etc/...
```bash
 network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.1.103/24]
      routes:
      - to: default
        via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

   ```

## **Instalación Nginx:**
* **Instalación de Nginx:**
```bash
   sudo apt update && sudo apt install nginx
   ```

* **Verificación del Servicio Nginx:**
```bash
   sudo systemctl status nginx
   ```

* **Instalación de módulos:**
```bash
 sudo apt install php8.3-fpm php8.3-mbstring php8.3-curl php8.3-xml php8.3-mysql php8.3-zip
```

## **node.js con nvm**

* **instalación**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

```bash
\. "$HOME/.nvm/nvm.sh"
```

```bash
node -v
```
respondera: v22.16.0

```bash
nvm current
```
respondera: v22.16.0

```bash
npm -v
```
respondera: 11.4.2

## **express js**

* **Creación de lacarpeta para la aplicación:**

```bash
mkdir myapp2
cd myapp2
```
* **Creación de la subcarpeta para el código de la API:**
Se la crea dentor de la carpeta myapp2
```bash
mkdir api
cd api/
```
* **Instalación de express js:**
```bash
npm init
```
* **Configuraciones:**

```bash
package name: (api)
version: (1.0.0)
description: API Backend SIS313
entry point: (index.js)
test command:
git repository:
keywords: API, SIS313
author: SIS313
license: (ISC)
About to write to /home/app2/myapp2/api/package.json:

{
  "name": "api",
  "version": "1.0.0",
  "description": "API Backend SIS313",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "API",
    "SIS313"
  ],
  "author": "SIS313",
  "license": "ISC"
}


Is this OK? (yes) y
```

* **Verificación:**
Dentro de /myapp1/api se ejecuta:
```bash
ls
```
tendria que verse: package.json, que es donde se guarda los archivos que permiten trabajar con nodejs y expressjs de la aplicación, usados en index.js
* **extensiones dentro del proyecto:**
```bash
npm install -g npm@11.4.2
```

## **prueba:**
Dentro de index.js se carga el código, entrando en:

```bash
nano index.js
```
Dentro se añade:
```bash
const express = require('express');
const app = express();
const PORT = 3001; // Puerto distinto para APP2

app.use(express.json());

app.get('/', (req, res) => {
  res.send('¡Bienvenido a la API SIS313 APP2!');
});

app.get('/usuarios', (req, res) => {
  res.json([
    { id: 1, nombre: 'Carlos' },
    { id: 2, nombre: 'Ana' }
  ]);
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor APP1 corriendo en http://localhost:${PORT});
});
```
Para hacer correr la API, se ejecuta:
```bash
node index.js
```
si todo salio bien, devuelve: Servidor API corriendo en http://localhost:3002

para verificar la pp2:
```bash
192.168.100.30:3002
```

con el balanceador:
```bash
sis313.final #dns del balanceador
```
# **Aplicación 2 con la bd**
# **Cliente de mariadb:**
* **extensiones de cliente para mariadb:**
```bash
npm install mysql2
```
# **CRUD BIBLIIOTECA de la app2:**
```bash
const express = require('express');
const mysql = require('mysql2');
const app = express();
const PORT = 3001;

app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Conexión con pool para MariaDB remota
const pool = mysql.createPool({
  host: '192.168.1.104',
  port: 3306,
  user: 'appcrud',
  password: 'CrudPass123!',
  database: 'biblioteca',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

app.get('/', (req, res) => {
  pool.query('SELECT * FROM libros', (err, libros) => {
    if (err) {
      console.error('Error consultando la base de datos:', err);
      return res.status(500).send('Error consultando la base de datos: ' + err.message);
    }

    let html = `
      <!DOCTYPE html>
      <html lang="es">
      <head>
        <meta charset="UTF-8" />
        <title>Libros - SIS313</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 40px; }
          table { border-collapse: collapse; width: 100%; margin-top: 20px; }
          th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
          th { background-color: #f2f2f2; }
          form input { padding: 6px; margin-right: 10px; }
          button { padding: 6px 12px; }
          .btn { cursor: pointer; }
          .btn-edit { color: blue; }
          .btn-delete { color: red; }
        </style>
      </head>
      <body>
        <h1>Libros - SIS313</h1>

        <form id="formAgregar">
          <input name="titulo" placeholder="Título del libro" required />
          <input name="autor" placeholder="Autor" required />
          <input name="anio_publicacion" type="number" placeholder="Año" required />
          <input name="precio" type="number" step="0.01" placeholder="Precio" required />
          <button type="submit">Agregar Libro</button>
        </form>

        <table id="tablaLibros">
          <thead>
            <tr>
              <th>ID</th><th>Título</th><th>Autor</th><th>Año</th><th>Precio</th><th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            ${libros.map(l => `
              <tr data-id="${l.id}">
                <td>${l.id}</td>
                <td class="titulo">${l.titulo}</td>
                <td class="autor">${l.autor}</td>
                <td class="anio">${l.anio_publicacion}</td>
                <td class="precio">$${Number(l.precio).toFixed(2)}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(${l.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(${l.id})">Borrar</button>
                </td>
              </tr>
            `).join('')}
          </tbody>
        </table>

        <script>
          const form = document.getElementById('formAgregar');
          form.addEventListener('submit', async e => {
            e.preventDefault();
            const formData = new FormData(form);
            const data = {
              titulo: formData.get('titulo'),
              autor: formData.get('autor'),
              anio_publicacion: parseInt(formData.get('anio_publicacion')),
              precio: parseFloat(formData.get('precio'))
            };
            const res = await fetch('/agregar', {
              method: 'POST',
              headers: {'Content-Type': 'application/json'},
              body: JSON.stringify(data)
            });
            if (res.ok) {
              form.reset();
              cargarLibros();
            } else {
              alert('Error al agregar libro');
            }
          });

          async function cargarLibros() {
            const res = await fetch('/libros');
            const libros = await res.json();
            const tbody = document.querySelector('#tablaLibros tbody');
            tbody.innerHTML = libros.map(l => \`
              <tr data-id="\${l.id}">
                <td>\${l.id}</td>
                <td class="titulo">\${l.titulo}</td>
                <td class="autor">\${l.autor}</td>
                <td class="anio">\${l.anio_publicacion}</td>
                <td class="precio">$\${Number(l.precio).toFixed(2)}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(\${l.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(\${l.id})">Borrar</button>
                </td>
              </tr>\`).join('');
          }

          async function borrar(id) {
            if (!confirm('¿Seguro que quieres borrar este libro?')) return;
            const res = await fetch('/borrar/' + id, { method: 'DELETE' });
            if (res.ok) {
              cargarLibros();
            } else {
              alert('Error al borrar libro');
            }
          }

          async function editar(id) {
            const tr = document.querySelector(\tr[data-id="\${id}"]\);
            const tituloTd = tr.querySelector('.titulo');
            const autorTd = tr.querySelector('.autor');
            const anioTd = tr.querySelector('.anio');
            const precioTd = tr.querySelector('.precio');

            const tituloOld = tituloTd.textContent;
            const autorOld = autorTd.textContent;
            const anioOld = anioTd.textContent;
            const precioOld = precioTd.textContent.replace('$','');

            tituloTd.innerHTML = \<input type="text" id="editTitulo" value="\${tituloOld}">\;
            autorTd.innerHTML = \<input type="text" id="editAutor" value="\${autorOld}">\;
            anioTd.innerHTML = \<input type="number" id="editAnio" value="\${anioOld}">\;
            precioTd.innerHTML = \<input type="number" step="0.01" id="editPrecio" value="\${precioOld}">\;

            const btnEdit = tr.querySelector('.btn-edit');
            btnEdit.textContent = 'Guardar';
            btnEdit.onclick = async () => {
              const nuevoTitulo = document.getElementById('editTitulo').value;
              const nuevoAutor = document.getElementById('editAutor').value;
              const nuevoAnio = parseInt(document.getElementById('editAnio').value);
              const nuevoPrecio = parseFloat(document.getElementById('editPrecio').value);
              const res = await fetch('/editar/' + id, {
                method: 'PUT',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({ titulo: nuevoTitulo, autor: nuevoAutor, anio_publicacion: nuevoAnio, precio: nuevoPrecio })
              });
              if (res.ok) {
                cargarLibros();
              } else {
                alert('Error al editar libro');
              }
            };
          }

          cargarLibros();
        </script>
      </body>
      </html>
    `;
    res.send(html);
  });
});

app.get('/libros', (req, res) => {
  pool.query('SELECT * FROM libros', (err, results) => {
    if (err) {
      console.error('Error en /libros:', err);
      return res.status(500).json({ error: err.message });
    }
    res.json(results);
  });
});

app.post('/agregar', (req, res) => {
  const { titulo, autor, anio_publicacion, precio } = req.body;
  pool.query(
    'INSERT INTO libros (titulo, autor, anio_publicacion, precio) VALUES (?, ?, ?, ?)',
    [titulo, autor, anio_publicacion, precio],
    (err) => {
      if (err) {
        console.error('Error en /agregar:', err);
        return res.status(500).json({ error: err.message });
      }
      res.status(200).json({ mensaje: 'Libro agregado' });
    }
  );
});

app.put('/editar/:id', (req, res) => {
  const id = req.params.id;
  const { titulo, autor, anio_publicacion, precio } = req.body;
  pool.query(
    'UPDATE libros SET titulo = ?, autor = ?, anio_publicacion = ?, precio = ? WHERE id = ?',
    [titulo, autor, anio_publicacion, precio, id],
    (err) => {
      if (err) {
        console.error('Error en /editar:', err);
        return res.status(500).json({ error: err.message });
      }
      res.status(200).json({ mensaje: 'Libro actualizado' });
    }
  );
});

app.delete('/borrar/:id', (req, res) => {
  const id = req.params.id;
  pool.query('DELETE FROM libros WHERE id = ?', [id], (err) => {
    if (err) {
      console.error('Error en /borrar:', err);
      return res.status(500).json({ error: err.message });
    }
    res.status(200).json({ mensaje: 'Libro borrado' });
  });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor API corriendo en http://localhost:${PORT});
});
```

si todo salio bien, devuelve: 
- Servidor API corriendo en http://localhost:3002
Conectado a la base de datos


## **Verificación**
para verificar con balanceador:
```bash
sis313.final 
```

</div>
