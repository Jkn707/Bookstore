# APLICACIÓN ESCALABLE
Este es un proyecto de una librería en línea (BookStore) desarrollado con Python y Flask, que utiliza HTML para la interfaz de usuario y está preparado para ejecutarse en contenedores mediante Docker.

## Miembros del equipo

| Nombre | Correo institucional | Rol |
|------|---------------------|------|
| Jose Alejandro Tordecilla | jatordeciz@eafit.edu.co | Desarrollador |
| Katherin Nathalia Allin Murillo | knallinm@eafit.edu.co | Desarrollador |
| Juan Andrés Montoya Galeano | jamontoya2@eafit.edu.co | Desarrolladora |

## Estructura del proyecto
```
Bookstore/
│
├── app.py                    # Punto de entrada principal de la app Flask
├── config.py                 # Configuraciones generales de la app
├── requirements.txt          # Dependencias del proyecto
├── Dockerfile                # Instrucciones para construir la imagen Docker
├── docker-compose.yml        # Orquestación de contenedores
│
├── controllers/              # Rutas y lógica del controlador
│   ├── __init__.py
│   └── book_controller.py
│
├── models/                   # Definición de modelos y ORM (SQLAlchemy)
│   ├── __init__.py
│   └── book.py
│
├── templates/                # Plantillas HTML para las vistas
│   ├── base.html
│   └── index.html
│
├── static/                   # Archivos estáticos (CSS, JS, imágenes)
│   ├── css/
│   └── js/
│
├── extensions.py             # Inicialización de extensiones Flask (ej. db, login)
└── README.md                 # Descripción del proyecto y guía de uso
```
Como podemos ver dentro del proyecto tenemos varios archivos pero vamos a resaltar un poco los que nos parecen más importantes:

* `app.py`: Es el archivo principal y se encarga de iniciar la aplicación Flask.
* `controllers/`: Contiene la lógica de las rutas y controladores de la aplicación.
* `models/`: Define las clases y estructuras de datos que representan las entidades del sistema.
* `templates/`: Incluye las plantillas HTML para la interfaz de usuario.
* `extensions.py`: Módulo que inicializa las extensiones de Flask.
* `config.py`: Archivo de configuración de la aplicación.
* `requirements.txt`: Lista de dependencias necesarias para ejecutar el proyecto.
* `Dockerfile` y `docker-compose.yml`: Archivos para la creación y gestión de contenedores Docker.

Entre las **tecnologías y recursos** que utilizamos para el desarrollo de este tenemos Python, Flask, HTML, Docker y Docker Compose, entre otros. Todo lo anterior nos permite llegar a el proyecto final donde se logra la gestión de los libros, la interfaz de usuario a través de la cual podemos gestionar estos recursos y el almacenamiento de la información más importante en base de datos.

## Paso a paso del despliegue de BookStore en la instancia EC2 de AWS

### 1. Preparación de la Máquina Virtual en AWS
#### 1.1. Iniciar sesión en AWS y lanzar una instancia EC2
- Inicia sesión en tu cuenta AWS Learner Lab
- Ve a la consola de EC2
- Haz clic en "Lanzar instancia"
- Configura tu instancia:
    - Nombre: BookStore-App
    - AMI: Ubuntu Server 22.04 LTS
    - Tipo de instancia: t2.micro (incluido en capa gratuita)
    - Par de claves: Crea un nuevo par de claves si no tienes uno
    - Configuración de red: Permitir tráfico SSH (puerto 22), HTTP (80) y HTTPS (443)
- Haz clic en "Lanzar instancia"
 
#### 1.2. Conectarse a la instancia EC2
- Actualizar el sistema
- `sudo apt update`
- `sudo apt upgrade -y`

### 2. Configuración del Dominio y DNS
#### 2.1. Apuntar tu dominio a la IP de AWS
- Colocar la IP pública de la máquina EC2 en la regla A de Hostinger.
  
#### 2.2. Verifica la propagación del DNS (puede tardar hasta 48 horas)
- Puedes usar una herramienta como dnschecker.org para verificar la propagación.
  
### 3. Instalación de Docker y Docker Compose
#### 3.1. Instalar Docker
- `sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`
- `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
- `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
- `sudo apt update sudo apt install docker-ce -y`
  
#### 3.2. Dar permisos al usuario
- `sudo usermod -aG docker ${USER}`
- Cierra la sesión y vuelve a conectarte para aplicar los cambios, esto lo haces usando el comando `exit`
- Vuelve a conectarte con SSH
  
#### 3.3. Instalar Docker Compose
- `sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
- `sudo chmod +x /usr/local/bin/docker-compose`
  
### 4. Instalación de NGINX y Certbot (para SSL)
#### 4.1. Instalar NGINX
- `sudo apt install nginx -y`
- `sudo systemctl enable nginx`
- `sudo systemctl start nginx`
  
#### 4.2. Instalar Certbot para certificado SSL
- `sudo apt install certbot python3-certbot-nginx -y`
  
### 5. Preparación y Despliegue de la Aplicación
#### 5.1. Crear directorio para la aplicación
- `mkdir -p ~/bookstore`
- `cd ~/bookstore`
  
#### 5.2. Subir los archivos de la aplicación
- `cd ~/bookstore`
- `git clone https://github.com/Jkn707/Bookstore`
  
#### 5.3. Modificar la configuración para producción
Edita el archivo config.py para usar MySQL en lugar de SQLite (ya está configurado correctamente para MySQL):
- `cd ~/bookstore`
- `cat config.py`
Verifica que SQLALCHEMY_DATABASE_URI apunte a MySQL

#### 5.4. Modificar el archivo app.py
Cambia la línea de ejecución en app.py para que no esté en modo debug:
- `nano app.py`
  
Cambia la última parte:
```
  if __name__ == '__main__':
       with app.app_context():
            db.create_all()
            initialize_delivery_providers()
       app.run(host="0.0.0.0", port=5000) # Elimina debug=True

```
#### 5.5. Crear un archivo .env para las variables de entorno (opcional)
- `nano .env`
  
Contenido:
```
     MYSQL_DATABASE=bookstore
     MYSQL_USER=bookstore_user
     MYSQL_PASSWORD=bookstore_pass
     MYSQL_ROOT_PASSWORD=root_pass
     FLASK_ENV=production
     SECRET_KEY=tu_clave_secreta_muy_segura

```
### 6. Configuración de NGINX como Proxy Inverso
#### 6.1. Crear configuración de NGINX para tu sitio
- `sudo nano /etc/nginx/sites-available/bookstore`
  
Contenido:
```
  server {
        listen 80;
        server_name laranatriste.site www.laranatriste.site;
        location / {
             proxy_pass http://localhost:5000;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
        }
  }
```
#### 6.2. Habilitar el sitio
- `sudo ln -s /etc/nginx/sites-available/bookstore /etc/nginx/sites-enabled/`
- `sudo nginx -t #Verificar la configuración`
- `sudo systemctl reload nginx`
  
### 7. Obtener Certificado SSL con Certbot
- `sudo certbot --nginx -d laranatriste.com -d www.laranatriste.com`
  
Sigue las instrucciones de Certbot para obtener y configurar automáticamente el certificado SSL.
Certbot modificará automáticamente la configuración de NGINX para redirigir HTTP a HTTPS y configurar el certificado SSL.

### 8. Arrancar la Aplicación con Docker Compose
- `cd ~/bookstore`
- `cd Bookstore`
- `docker-compose up -d`
  
#### 8.1. Verificar el estado de los contenedores
- `docker-compose ps`
  
#### 8.2. Verificar los logs para detectar posibles errores
- `docker-compose logs -f`
  
### Visitar [https://laranatriste.site](https://laranatriste.site) en el navegador.
  

## Instrucciones de uso
Para iniciar la base de datos y crear las tablas:
* `flask shell`
Esto es para iniciar el shell de Flask, posteriormente para inicializar la base de datos y crear las tablas definidas en los modelos vamos a ejecutar el siguiente comando:
* `from extensions import db`
* `db.create_all()`
