# Laboratorio 4.3: Instalación y Configuración de Nginx con PHP-FPM en Ubuntu Server 24.04 LTS



**Equipamiento Necesario:**



**Pasos:**

**1. Instalación de Nginx:**


   ```bash
   sudo apt update && sudo apt install nginx
   ```

**2. Verificación del Estado del Servicio Nginx:**

   
   ```bash
   sudo systemctl status nginx
   ```


**3. Configuración Básica del Firewall (UFW):**

   Si UFW está habilitado, permite el tráfico HTTP (puerto 80) y HTTPS (puerto 443) para Nginx con los comandos:
   ```bash
   sudo ufw allow 'Nginx HTTP'
   ```
   Y para habilitar HTTPS:
   ```bash
   sudo ufw allow 'Nginx HTTPS'
   ```
   O para habilitar ambos (HTTP y HTTPS) utiliza:
   ```bash
   sudo ufw allow 'Nginx Full'
   ```
   Verifica el estado del firewall con:
   ```bash
   sudo ufw status
   ```

**4. Acceso a la Página Web Predeterminada de Nginx:**


**5: Exploración de la Configuración de Nginx**
