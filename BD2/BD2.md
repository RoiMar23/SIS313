# Configuración de Servidor Esclavo - MariaDB (192.168.1.105)

## Paso 1: Configurar el archivo del servidor
Editar el archivo:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Agregar al final:
```ini
[mysqld]
server-id=2
relay-log=mariadb-relay-bin
```

## Paso 2: Reiniciar MariaDB
```bash
sudo systemctl restart mariadb
```

## Paso 3: Transferir el archivo `biblioteca.sql` desde el maestro
```bash
scp -P 2205 biblioteca.sql usuario@192.168.1.105:~
```

## Paso 4: Importar el volcado SQL
```bash
mysql -u root < biblioteca.sql
```

## Paso 5: Conectar con el maestro
En MariaDB:
```sql
STOP SLAVE;

CHANGE MASTER TO
  MASTER_HOST='192.168.1.104',
  MASTER_USER='replica_user',
  MASTER_PASSWORD='clave_segura',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=328;

START SLAVE;
```

## Paso 6: Verificar replicación
```sql
SHOW SLAVE STATUS\G;
```
Debes ver:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Last_IO_Error:` vacío

## Paso 7: Probar replicación
Insertar un registro en el maestro:
```sql
INSERT INTO libros (titulo, autor, ano, publicacion, precio)
VALUES ('Cien años de soledad', 'Gabriel García Márquez', 1967, 'Sudamericana', 45.00);
```
Verificar en el esclavo:
```sql
SELECT * FROM libros;
```
El nuevo registro debe aparecer automáticamente.
