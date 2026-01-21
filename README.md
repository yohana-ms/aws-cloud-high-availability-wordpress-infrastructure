# AWS Cloud Infrastructure: High Availability WordPress
> **Nota Personal:** Este repositorio documenta mi proceso de aprendizaje y despliegue de una arquitectura robusta en la nube usando AWS. El objetivo es entender cómo se conectan las piezas de infraestructura (Red, Cómputo y Datos) sin depender de configuraciones por defecto.

## 🎯 Objetivo del Proyecto
Crear un sitio web (WordPress) que sea:
1.  **Seguro:** La base de datos no es accesible desde internet.
2.  **Resiliente:** Si el servidor se apaga y lo vuelvo a prender, la data no se pierde (porque está en RDS).
3.  **Automatizado:** El servidor se configura solo al iniciarse usando scripts.

---

## 🛠️ Servicios Utilizados (¿Qué hace cada uno?)

### 1. VPC (Virtual Private Cloud) - "La Red"
No usé la red por defecto. Creé una **VPC personalizada** (`proyecto-cloud-vpc`) para entender la segmentación de redes.
* **Subredes Públicas:** Donde viven los servidores web (EC2) para que los usuarios puedan entrar.
* **Subredes Privadas:** Donde vive la Base de Datos (RDS) para que nadie la ataque desde fuera.

### 2. EC2 (Elastic Compute Cloud) - "El Servidor"
Usé instancias `t2.micro` con sistema operativo **Ubuntu**.
* **Bootstrapping (User Data):** En lugar de instalar Apache y PHP comando por comando manualmente cada vez, escribí un script en Bash (`setup_script.sh`) que se ejecuta automáticamente cuando la instancia nace.
* **Aprendizaje clave:** Automatizar la instalación reduce errores humanos.

### 3. Amazon RDS (Relational Database Service) - "Los Datos"
En lugar de instalar MySQL dentro del servidor (que es mala práctica), usé RDS.
* **Motor:** MySQL Community.
* **Beneficio:** AWS maneja los parches de seguridad y backups.
* **Conexión:** Configuré la seguridad para que la RDS *solo* acepte conexiones que vengan desde mi servidor Web (EC2), bloqueando todo lo demás.

### 4. Security Groups - "El Firewall Virtual"
Configuré reglas de "Mínimo Privilegio":
* `web-sg`: Permite tráfico HTTP (80) desde cualquier lugar (0.0.0.0/0).
* `rds-sg`: Permite tráfico MySQL (3306) **SOLO** desde el `web-sg`.

---

## 🚀 Guía de Despliegue (Paso a Paso)

### Paso 1: Infraestructura de Red
Crear la VPC con 2 subredes públicas y 2 privadas en distintas zonas de disponibilidad (AZ) para preparar la Alta Disponibilidad.

### Paso 2: Base de Datos
Lanzar RDS MySQL en la subred privada.
* *Nota:* Guardar bien el "Endpoint" (URL de la BD), el usuario y la contraseña.

### Paso 3: Servidor Web (EC2)
Lanzar instancia Ubuntu en subred pública.
* Pegar el script `setup_script.sh` en la sección "User Data".
* Asegurar que tenga IP pública habilitada.

### Paso 4: Conexión Final (El momento de la verdad)
1.  Tomar la IP Pública de la EC2.
2.  Pegarla en el navegador.
3.  Configurar WordPress con las credenciales del RDS.

---

## 📂 Archivos en este repositorio
* `setup_script.sh`: El script de Bash que instala Apache, PHP y descarga WordPress.
* `evidencia/`: Capturas de pantalla que demuestran la configuración de la VPC y el RDS funcionando.
