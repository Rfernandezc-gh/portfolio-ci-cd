# 🏗️ Documentación de Infraestructura: CI/CD Blindado

## 1. Presentación
Este proyecto implementa un pipeline de **Integración y Despliegue Continuo (CI/CD)** automatizado en github actions. Su función principal es detectar cualquier cambio en el código fuente, empaquetarlo en una imagen ligera y desplegarlo en un servidor remoto de forma segura y sin intervención manual.

Aunque este flujo está optimizado para **Portainer** sobre **Oracle Cloud**, su lógica basada en contenedores e imágenes es totalmente compatible con entornos de **Kubernetes** y cualquiero otro tipo de **Nube** lo que lo hace una solución escalable y profesional.

> **¿Por qué este proyecto?** En lugar de subir archivos por FTP o entrar por SSH, este sistema garantiza que el servidor siempre tenga la versión exacta del código que está en la rama principal, eliminando el error humano, ademas de tener un control de versiones por github.

---

## 2. Tecnologías y Estructura
Para que este sistema sea robusto, se han integrado herramientas líderes en la industria:

### Stack Tecnológico
| Herramienta | Función |
| :--- | :--- |
| **GitHub Actions** | Orquestador de las tareas de automatización. |
| **Docker & Buildx** | Creación de imágenes multiplataforma (Soporte nativo para ARM64/Oracle). |
| **Docker Hub** | Registro centralizado de versiones de la aplicación. |
| **Portainer** | Agente de despliegue y gestión de contenedores en el host. |
| **Cloudflare Zero Trust** | Túnel de seguridad que blinda el acceso al servidor. |

---

## 3. Procedimiento Paso a Paso y Lógica Técnica

Para conseguir un despliegue 100% automatizado y seguro, la infraestructura se ha configurado en las siguientes 4 fases:

### Fase A: El Receptor (Portainer Webhook)
Lo primero es hacer que el servidor tenga un oido para que sepa cuándo debe actualizar un contenedor o stack. Para ello, activamos la opción de **Service Webhook** dentro de la configuración del Stack en Portainer. Esto genera una URL única.
* **¿Por qué se hace esto?** En infraestructuras tradicionales, actualizar implica conectarse al servidor por SSH, descargar la nueva imagen y reiniciar los servicios manualmente. El Webhook automatiza todo este proceso: al recibir una simple petición, Portainer hace todo el trabajo pesado de forma desatendida.* 


> **<img width="916" height="515" alt="Screenshot 2026-05-07 102221" src="https://github.com/user-attachments/assets/4097e8eb-c619-4ab0-a571-454b2fe9a9a6" />



### Fase B: El Portero de Seguridad (Cloudflare Zero Trust)
Tener una URL de Webhook expuesta a internet es un riesgo de seguridad crítico. Para solucionarlo, colocamos la ruta del Webhook (`/api/stacks/webhooks/*`) detrás de una política de **Cloudflare Zero Trust**. Creamos unas credenciales para máquinas llamadas **Service Tokens** (`Client ID` y `Client Secret`).
* **¿Por qué se hace esto?** Para implementar una arquitectura de "Confianza Cero". Si un atacante descubre la URL del Webhook e intenta dispararlo, Cloudflare le bloqueará el acceso al instante. Solo nuestro pipeline, que posee las llaves criptográficas ocultas, puede atravesar el túnel y llegar al servidor.

> **📸 [INSERTA AQUÍ LA CAPTURA]:** Panel de Cloudflare Zero Trust mostrando la regla de acceso (Service Auth) vinculada al Webhook.

### Fase C: Evasión de Sistemas Antibots (WAF Bypass)
Cloudflare cuenta con un **Bot Fight Mode** que bloquea peticiones automatizadas (como las que hace GitHub) lanzando un desafío Captcha ("Just a moment..."). Para que nuestro pipeline no se quede atascado en este paso, creamos una **Regla WAF (Custom Rule)** con la acción **Skip** (Omitir) exclusivamente para la ruta del Webhook.
* **¿Por qué se hace esto?** Porque `curl` (la herramienta que usa GitHub para llamar al servidor) no es un navegador humano y no puede resolver Captchas. Al omitir el WAF solo en esta ruta, permitimos que la automatización llegue a la puerta de Zero Trust sin ser bloqueada por el camino, manteniendo el resto de la web protegida contra bots.

> **📸 [INSERTA AQUÍ LA CAPTURA]:** Regla del WAF de Cloudflare configurada como 'Skip' para el path del webhook, desactivando el Bot Fight Mode.

### Fase D: El Motor de Construcción (GitHub Actions)
Finalmente, orquestamos todo en el archivo `.github/workflows/deploy.yml`. Este script automatiza el flujo completo cada vez que se hace un `git push` a la rama `main`:
1.  **Setup QEMU & Buildx:** Prepara el entorno para compilar la imagen en arquitectura **ARM64** (requerido por servidores como Oracle Cloud Ampere).
2.  **Build & Push:** Empaqueta el código en una imagen Docker y la sube al registro de Docker Hub.
3.  **Trigger Webhook:** Ejecuta el comando `curl` disfrazado con un *User-Agent* de navegador, adjuntando las llaves de Zero Trust para activar la actualización en Portainer.
* **¿Por qué se hace esto?** Es el núcleo de la filosofía CI/CD (Integración y Despliegue Continuo). Garantiza que el código de producción sea exactamente el mismo que el del repositorio, compilado en un entorno limpio e inmutable.

> **📸 [INSERTA AQUÍ LA CAPTURA]:** Código YAML del pipeline o una captura de GitHub Actions mostrando los pasos completados con éxito (en verde).
