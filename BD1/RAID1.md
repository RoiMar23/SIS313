# Configuración de RAID 1 para el Servidor Maestro (MariaDB)

## Objetivo

**Objetivo del Maestro**: Implementar RAID 1 en el servidor maestro de MariaDB para garantizar **alta disponibilidad** y **resiliencia frente a fallos de disco**. El RAID 1 permite continuar con el servicio incluso si uno de los discos físicos falla, además de mantener replicación hacia el esclavo y asegurar integridad de los datos.

**Objetivo del RAID 1**: Asegurar que los datos escritos en el maestro se dupliquen automáticamente en otro disco (espejo) para prevenir pérdida ante fallos. Además, mantener backups de la base `biblioteca` dentro del RAID para una restauración rápida.

---

## Entorno

- Sistema operativo: Ubuntu Server 24.04
- Base de datos: MariaDB 10.11
- Discos: 2 discos de almacenamiento de igual capacidad (ej. `/dev/sdb` y `/dev/sdc`)

---

## Pasos para configurar RAID 1 en el servidor Maestro

### 1. Verificar discos disponibles

```bash
lsblk
```

Busca dos discos **sin particiones activas** (ej. `/dev/sdb` y `/dev/sdc`).

### 2. Instalar `mdadm`

```bash
sudo apt update
sudo apt install mdadm -y
```

### 3. Crear el arreglo RAID 1

```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

### 4. Formatear el volumen RAID

```bash
sudo mkfs.ext4 /dev/md0
```

### 5. Montar el volumen RAID

```bash
sudo mkdir -p /mnt/raid1
sudo mount /dev/md0 /mnt/raid1
```

### 6. Configurar montaje automático

Obtener UUID:

```bash
sudo blkid /dev/md0
```

Editar fstab:

```bash
sudo nano /etc/fstab
```

Agregar la línea:

```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/raid1 ext4 defaults,nofail,discard 0 0
```

### 7. Mover la base de datos a RAID

> Realiza un **backup** primero.

```bash
sudo systemctl stop mariadb
sudo cp -r /var/lib/mysql /mnt/raid1/
sudo mv /var/lib/mysql /var/lib/mysql.bak
sudo ln -s /mnt/raid1/mysql /var/lib/mysql
sudo chown -R mysql:mysql /mnt/raid1/mysql
sudo systemctl start mariadb
```

---

## Realizar Backup de la Base de Datos en RAID

### 1. Crear carpeta de backups (si no existe)

```bash
sudo mkdir -p /mnt/raid1/backups
```

### 2. Ejecutar el backup

```bash
sudo sh -c 'mysqldump -u root --databases biblioteca > /mnt/raid1/backups/biblioteca.sql'
```

### 3. Verificar el backup

```bash
ls -lh /mnt/raid1/backups/
cat /mnt/raid1/backups/biblioteca.sql | head
```

Este archivo contiene el volcado SQL de toda la base de datos `biblioteca`.

---

## Simulación de Falla en RAID 1

### 1. Ver estado del RAID

```bash
cat /proc/mdstat
```

### 2. Simular fallo en un disco

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb
sudo mdadm --manage /dev/md0 --remove /dev/sdb
```

### 3. Confirmar que RAID sigue funcionando con un solo disco

```bash
cat /proc/mdstat
```

### 4. Restaurar el disco dañado (simulado)

```bash
sudo mdadm --add /dev/md0 /dev/sdb
```

Podrás observar un proceso de recuperación:

```bash
cat /proc/mdstat
```

Ejemplo:

```
md0 : active raid1 sdb[2] sdc[1]
      10476544 blocks super 1.2 [2/1] [_U]
      [===>.................]  recovery = 15.2% (1602048/10476544) finish=0.6min speed=228864K/sec
```

Esto significa que el RAID está recuperándose:

- `[_U]`: indica que el disco `sdb` fue marcado como fallido y está siendo reconstruido.
- El progreso se muestra con `recovery = X%`.

Cuando el proceso termine, verás:

```
[2/2] [UU]
```

Ambos discos estarán sincronizados nuevamente.

---

## Efecto en la replicación MariaDB (Maestro - Esclavo)

### Si RAID 1 está funcionando:

- La replicación continúa normalmente desde el maestro.
- El esclavo no detecta cambios ya que el maestro sigue disponible.

### Si RAID no está y falla el disco maestro:

- La base se pierde si no hay respaldo.
- El esclavo no puede seguir replicando.
- El RAID 1 previene este escenario permitiendo mantener la replicación activa.

---

## Conclusión

RAID 1 protege la base de datos del maestro ante fallos físicos, garantizando la continuidad de la replicación hacia el esclavo.

---
