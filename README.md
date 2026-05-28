# 🚀 Actividad Práctica — Instalación de n8n y Primer "Hola Mundo"
 
**Integrante:** Kevin andres castillo pabon  
**Fecha:** 27/05/2026  
**Método utilizado:** Opción 2 — Docker con ngrok (Self-Hosted)
 
---
 
## 📋 Descripción General
 
Este proyecto documenta la instalación de **n8n self-hosted** utilizando Docker Desktop y ngrok en Windows, junto con la creación de un primer workflow funcional tipo "Hola Mundo". n8n es una plataforma de automatización de flujos de trabajo de código abierto que permite conectar APIs, servicios y herramientas mediante una interfaz visual basada en nodos.
 
---
 
## 🛠️ Requisitos Previos
 
| Herramienta | Versión | Descripción |
|---|---|---|
| Windows | 10/11 | Sistema operativo utilizado |
| Docker Desktop | 29.5.2 (build 79eb04c) | Motor de contenedores |
| Docker Compose | v5.1.3 | Orquestador de servicios |
| VS Code | Última estable | Editor de código |
| Cuenta ngrok | Gratuita | Túnel HTTPS público |
 
---
 
## ⚙️ Proceso de Instalación
 
### Paso 1 — Instalación de Docker Desktop
 
Se descargó Docker Desktop desde [docker.com](https://www.docker.com/products/docker-desktop) seleccionando la versión **Windows AMD64**, ya que el equipo utiliza un procesador Intel/AMD x64.
 
Se ejecutó el instalador y se siguió el asistente de configuración. Al finalizar, se reinició el equipo y se verificó la instalación ejecutando los siguientes comandos en la terminal:
 
```bash
docker --version
# Docker version 29.5.2, build 79eb04c
 
docker compose version
# Docker Compose version v5.1.3
```
 
Ambos comandos devolvieron número de versión correctamente ✅
 
---
 
### Paso 2 — Configuración de ngrok
 
Se creó una cuenta gratuita en [ngrok.com](https://ngrok.com) y se obtuvieron dos datos esenciales desde el panel de control:
 
- **Dominio gratuito** (sección *Domains*): `cameo-bannister-pupil.ngrok-free.dev`
- **Token de autenticación** (sección *Your Authtoken*): valor personal y privado
ngrok permite exponer la instancia local de n8n a internet mediante un túnel HTTPS seguro, lo cual es necesario para que los webhooks externos puedan comunicarse con n8n.
 
---
 
### Paso 3 — Creación del archivo docker-compose.yml
 
Se creó una carpeta llamada `n8n_actividad` en el escritorio y dentro de ella se creó el archivo `docker-compose.yml` con el siguiente contenido:
 
```yaml
version: '3.8'
 
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=cameo-bannister-pupil.ngrok-free.dev
      - WEBHOOK_URL=https://cameo-bannister-pupil.ngrok-free.dev
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
    volumes:
      - n8n_data:/home/node/.local/share/n8n
 
  ngrok:
    image: ngrok/ngrok:latest
    restart: always
    environment:
      - NGROK_AUTHTOKEN=[TOKEN_PRIVADO]
    command:
      - "http"
      - "n8n:5678"
      - "--domain=cameo-bannister-pupil.ngrok-free.dev"
    depends_on:
      - n8n
 
volumes:
  n8n_data:
```
 
**Explicación del archivo:**
- `n8n`: contenedor principal que ejecuta el motor de automatización en el puerto 5678
- `ngrok`: contenedor que crea el túnel público HTTPS hacia n8n
- `volumes`: garantiza que los flujos y datos persistan aunque los contenedores se reinicien
- `depends_on`: ngrok espera a que n8n arranque primero
- `restart: always`: reinicia automáticamente si el servicio falla
---
 
### Paso 4 — Levantar los contenedores
 
Desde la terminal integrada de VS Code (Ctrl + `) se ejecutó:
 
```bash
docker compose up -d
```
 
El parámetro `-d` ejecuta los contenedores en segundo plano (*detached mode*). Docker descargó automáticamente las imágenes de n8n y ngrok desde Docker Hub.
 
**Resultado:**
```
✔ Image ngrok/ngrok:latest     Pulled
✔ Image n8nio/n8n:latest       Pulled
✔ Network n8n_actividad_default   Created
✔ Volume n8n_actividad_n8n_data   Created
✔ Container n8n_actividad-n8n-1   Started
✔ Container n8n_actividad-ngrok-1 Started
```
 
Para verificar que ambos contenedores estaban corriendo:
 
```bash
docker compose ps
```
 
```
✔ Container n8n_actividad-n8n-1    Running
✔ Container n8n_actividad-ngrok-1  Running
```
 
---
 
### Paso 5 — Acceso a n8n
 
Se abrió el navegador y se ingresó al dominio:
 
```
https://cameo-bannister-pupil.ngrok-free.dev
```
 
n8n mostró la pantalla de configuración inicial donde se creó la cuenta de administrador (owner). Tras completar el registro se accedió al dashboard principal de n8n con HTTPS activo y webhooks funcionales ✅
 
---
 
## ⚠️ Problemas Encontrados y Soluciones
 
| # | Problema | Solución |
|---|---|---|
| 1 | `Docker Desktop is unable to start` al ejecutar `docker compose up -d` | Docker Desktop no estaba corriendo en segundo plano. Se abrió manualmente y se esperó a que el ícono de la ballena quedara estable antes de volver a ejecutar el comando |
| 2 | `WEBHOOK_URL` sin `https://` en el archivo docker-compose.yml | Se corrigió agregando `https://` al inicio: `WEBHOOK_URL=https://cameo-bannister-pupil.ngrok-free.dev` |
 
---
 
## 🔄 Workflow "Hola Mundo"
 
### Descripción
 
Se construyó un workflow básico compuesto por dos nodos conectados en secuencia que, al ejecutarse manualmente, genera un mensaje de "Hola Mundo".
 
**Nombre del workflow:** `hola mundo-kevin`
 
---
 
### Nodos utilizados
 
#### Nodo 1 — Manual Trigger (Disparador Manual)
- **Tipo:** Trigger
- **Función:** Inicia el workflow cuando el usuario hace clic en el botón "Execute workflow"
- **Configuración:** Sin configuración adicional requerida
- **Cuándo usarlo:** Para pruebas y ejecuciones manuales donde no se necesita un disparador automático
#### Nodo 2 — Code in JavaScript
- **Tipo:** Procesamiento
- **Función:** Ejecuta código JavaScript y genera el objeto de respuesta con el mensaje "Hola Mundo"
- **Lenguaje:** JavaScript
- **Modo:** Run Once for All Items
**Código utilizado:**
```javascript
return [
  {
    json: {
      mensaje: "Hola Mundo desde n8n",
      autor: "Kevin",
      estado: "Mi primer workflow funciona correctamente"
    }
  }
];
```
 
---
 
### Resultado del Output
 
Al ejecutar el workflow se obtuvo el siguiente resultado en el panel OUTPUT:
 
| mensaje | autor | estado |
|---|---|---|
| Hola Mundo desde n8n | Kevin | Mi primer workflow funciona correctamente |
 
El mensaje **"Workflow executed successfully"** confirmó la ejecución exitosa ✅
 
---
 
## 🏗️ Arquitectura del Sistema
 
```
Usuario
   ↓
https://cameo-bannister-pupil.ngrok-free.dev  (ngrok - túnel HTTPS)
   ↓
localhost:5678  (n8n - motor de automatización)
   ↓
Docker (orquesta ambos contenedores)
```
 
---
 
## 💭 Reflexión Técnica
 
**1. ¿Qué fue lo más difícil de la instalación?**
 
Lo más difícil fue entender que Docker Desktop debía estar corriendo en segundo plano antes de ejecutar cualquier comando. Al principio al ejecutar `docker compose up -d` aparecía el error `Docker Desktop is unable to start` porque la aplicación no estaba abierta. Una vez entendido ese punto, todo fluyó sin problemas. También fue importante no olvidar el `https://` en la variable `WEBHOOK_URL`.
 
**2. ¿Qué ventajas tiene n8n?**
 
n8n tiene varias ventajas importantes: es de código abierto y gratuito en su versión self-hosted, lo que da control total sobre los datos. Su interfaz visual basada en nodos permite construir automatizaciones complejas sin necesidad de escribir mucho código. Además tiene integraciones nativas con cientos de servicios como Telegram, OpenAI, Gmail, bases de datos y APIs REST, lo que lo hace muy versátil para proyectos reales.
 
**3. ¿Qué diferencia encontraron entre Docker y local?**
 
Con Docker no es necesario instalar Node.js ni gestionar dependencias manualmente en el sistema operativo. Todo corre dentro de contenedores aislados, lo que evita conflictos con otras aplicaciones. Además, el archivo `docker-compose.yml` hace que el entorno sea completamente reproducible: se puede copiar a cualquier máquina y funcionará exactamente igual. La instalación local con npm es más rápida de configurar inicialmente, pero Docker es más profesional, estable y cercano a un entorno de producción real.
 
**4. ¿Para qué casos reales usarían automatización?**
 
La automatización con n8n sería muy útil para notificaciones automáticas cuando ocurre un evento en una base de datos, para enviar reportes periódicos por correo o Telegram, para integrar formularios web con hojas de cálculo o CRMs, y para crear bots de atención al cliente que respondan automáticamente según reglas definidas.
 
**5. ¿Qué les gustaría automatizar en el futuro?**
 
Me gustaría automatizar un sistema que monitoree precios de productos en tiendas online y envíe una alerta por Telegram cuando baje el precio de algo específico. También sería interesante crear un agente con IA que clasifique y responda correos automáticamente según su contenido.
 
---
 
## 📁 Estructura del Repositorio
 
```
n8n_actividad/
├── docker-compose.yml
└── README.md
```
 
---
 
## 🔧 Comandos de Referencia
 
```bash
# Levantar los contenedores en segundo plano
docker compose up -d
 
# Ver el estado de los contenedores
docker compose ps
 
# Ver los logs en tiempo real
docker compose logs -f
 
# Detener los contenedores
docker compose down
 
# Actualizar las imágenes
docker compose pull
```
 