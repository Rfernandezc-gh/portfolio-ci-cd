#  Documentación de Infraestructura: CI/CD Blindado

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
* **¿Por qué se hace esto?** Para implementar una arquitectura segura, Si un atacante descubre la URL del Webhook e intenta dispararlo, Cloudflare le bloqueará el acceso al instante. Solo nuestro pipeline, que posee las llaves criptográficas ocultas, puede atravesar el túnel y llegar al servidor.

> **<img width="1048" height="229" alt="image" src="https://github.com/user-attachments/assets/aa6796e9-643c-446f-b744-252c52dd0d9f" />


### Fase C: El Motor de Construcción y Autenticación (GitHub Actions)
Finalmente, orquestamos todo en el archivo `.github/workflows/deploy.yml`. Este script automatiza el flujo completo cada vez que se hace un `git push` a la rama `main`:
1.  **Setup QEMU & Buildx:** Prepara el entorno para compilar la imagen en arquitectura **ARM64** (requerido por servidores como Oracle Cloud Ampere; si se despliega en otro proveedor, habría que investigar qué arquitectura usan).
2.  **Autenticación y Push (Docker Hub):** El pipeline inicia sesión en Docker Hub utilizando un **Access Token** seguro. A continuación, empaqueta el código en una imagen Docker y la sube a un registro **privado**.
3.  **Trigger Webhook:** Ejecuta el comando `curl` disfrazado con un *User-Agent* de navegador, adjuntando las llaves de Zero Trust para ordenar a Portainer que inicie la actualización. (Para que esto funcione, Portainer también ha sido configurado con su propio Access Token para poder descargar la imagen).
* **¿Por qué usar Access Tokens en vez de contraseñas?** Al mantener la imagen de Docker privada (vital para no exponer el código fuente a internet), tanto GitHub (para subirla) como Portainer (para descargarla) necesitan autenticarse de alguna manera. Utilizar **Access Tokens** independientes en lugar de la contraseña principal aplica el principio de seguridad de *Mínimo Privilegio*. Si alguno de los dos entornos se viera comprometido, puedes revocar ese token específico al instante sin perder tu cuenta de Docker ni afectar a otros servicios.

> **<img width="1434" height="761" alt="image" src="https://github.com/user-attachments/assets/40dbaaff-4665-4761-a86c-c81c387a8596" />


---

##  Por qué esta arquitectura es "Enterprise Ready"

1. **Eficiencia de Costes:** El pipeline está diseñado para correr sobre la capa gratuita de Oracle Cloud y Docker Hub, eliminando costes de infraestructura innecesarios.
2. **Seguridad Multi-capa:** La seguridad no depende solo de una contraseña, sino de una combinación de Túneles, Service Tokens y ocultación de la IP real del servidor.
3. **Escalabilidad:** Aunque actualmente despliega un portfolio, el mismo flujo es capaz de gestionar microservicios complejos con solo duplicar los pasos de compilación.
4. **Mantenibilidad:** El uso de imágenes inmutables permite realizar un **Rollback** instantáneo a la versión anterior si algo falla en producción, simplemente cambiando el tag de la imagen en Portainer.
   
---

##  5. Decisiones Técnicas y Criterios de Diseño

En el desarrollo de esta infraestructura se han tomado decisiones estratégicas para garantizar un entorno de nivel profesional, priorizando la seguridad y la eficiencia de costes:

###  Seguridad: Zero Trust vs. Exposición de Puertos
A diferencia de los despliegues básicos donde se abren puertos en el firewall (exponiendo el servidor a ataques de fuerza bruta), este proyecto implementa **Cloudflare Zero Trust**.
* **Criterio:** "Invisible al público". El panel de gestión (Portainer) no tiene una IP pública expuesta.
* **Service Tokens:** Se han sustituido las contraseñas tradicionales por **tokens de servicio**. Esto permite que el pipeline tenga acceso "Headless" (sin humano) de forma segura.

###  Infraestructura Inmutable
El pipeline sigue estrictamente el concepto de infraestructura inmutable.
* **Criterio:** Nunca se modifica el código directamente dentro del contenedor. 
* **Beneficio:** Cada despliegue es una copia limpia y nueva. Esto permite realizar un **Rollback** (volver atrás) instantáneo simplemente seleccionando el tag de la imagen anterior en Docker Hub si algo falla en producción.

###  Gestión de Secretos y Cumplimiento
Ninguna credencial, token o ID está escrito en el código fuente.
* **Implementación:** Se utiliza el almacén cifrado de **GitHub Secrets**. Esto demuestra el cumplimiento con las normativas de seguridad actuales, evitando filtraciones accidentales de credenciales en el historial de Git.

