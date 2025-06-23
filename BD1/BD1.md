# Configuración de Servidor Maestro - MariaDB (192.168.1.104)

## Paso 1: Configurar el archivo del servidor
Editar el archivo:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Agregar al final:
```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-do-db=biblioteca
```

## Paso 2: Reiniciar el servicio MariaDB
```bash
sudo systemctl restart mariadb
```

## Paso 3: Crear usuario de replicación
```sql
CREATE USER 'replica_user'@'%' IDENTIFIED BY 'clave_segura';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
```

## Paso 4: Crear base de datos y tabla de ejemplo
```sql
CREATE DATABASE biblioteca;
USE biblioteca;

CREATE TABLE libros (
  id INT AUTO_INCREMENT PRIMARY KEY,
  titulo VARCHAR(100),
  autor VARCHAR(100),
  ano INT,
  publicacion VARCHAR(100),
  precio DECIMAL(10,2)
);

INSERT INTO libros (titulo, autor, ano, publicacion, precio) VALUES
('1984', 'George Orwell', 1949, 'Secker & Warburg', 50.00);
```

## Paso 5: Crear un volcado (`.sql`) con información de replicación
```bash
mysqldump -u root --databases biblioteca --master-data --single-transaction > biblioteca.sql
```

Buscar en el archivo el bloque:
```sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=328;
```
Guardar esta información.
