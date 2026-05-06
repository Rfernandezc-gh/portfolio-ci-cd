# 🚀 Portfolio Pro: Arquitectura CI/CD Inmutable y Blindada

Este proyecto no es solo un portfolio; es una infraestructura completa de despliegue continuo (CI/CD) diseñada para ser **segura, eficiente y automatizada**. El sistema se encarga de compilar, subir y actualizar la aplicación en un servidor remoto (Oracle Cloud) sin intervención humana.



## 🏗️ Arquitectura del Sistema

El flujo de despliegue sigue una estrategia de **Infraestructura Inmutable**:

1.  **Code Push**: Al subir cambios a la rama `main`, se dispara un flujo en **GitHub Actions**.
2.  **Multi-Arch Build**: GitHub compila una imagen Docker optimizada para **ARM64** (arquitectura de los servidores Oracle Cloud).
3.  **Registry**: La imagen se almacena en **Docker Hub**.
4.  **Secure Webhook**: GitHub envía una señal cifrada a **Portainer** para indicar que hay una nueva versión disponible.
5.  **Auto-Update**: Portainer detiene el contenedor antiguo, descarga la nueva imagen y levanta el servicio en milisegundos.

## 🛡️ Seguridad y Blindaje (Cloudflare Zero Trust)

Para evitar que el servidor esté expuesto a internet, se ha implementado un esquema de seguridad de "Confianza Cero":

*   **Service Tokens**: El acceso al Webhook de Portainer está protegido por Cloudflare Zero Trust. Solo GitHub Actions posee las llaves (`Client ID` y `Client Secret`) para pasar.
*   **WAF Skip Rules**: Se ha configurado una regla personalizada en el Firewall de Cloudflare para omitir el "Bot Fight Mode" exclusivamente en la ruta del Webhook, permitiendo que la automatización pase sin enfrentar desafíos de Captcha.
*   **User-Agent Masking**: La petición de despliegue utiliza un User-Agent de navegador moderno para integrarse de forma natural con las políticas de tráfico de la red.

## 🛠️ Tecnologías Utilizadas

| Componente | Tecnología |
| :--- | :--- |
| **Runtime** | Docker & Docker Compose |
| **Orquestación** | Portainer |
| **CI/CD** | GitHub Actions |
| **Infraestructura** | Oracle Cloud (ARM64) |
| **Seguridad/Red** | Cloudflare Zero Trust & WAF |

## ⚙️ Configuración del Pipeline

El archivo de configuración se encuentra en `.github/workflows/deploy.yml`. Para que funcione, es necesario configurar los siguientes **Secrets** en el repositorio de GitHub:

*   `DOCKER_USER`: Usuario de Docker Hub.
*   `DOCKER_PASSWORD`: Token de acceso de Docker Hub.
*   `CF_CLIENT_ID`: ID del Service Token de Cloudflare.
*   `CF_CLIENT_SECRET`: Secret del Service Token de Cloudflare.

## 📋 Comandos de Mantenimiento

Si necesitas disparar el despliegue manualmente desde tu terminal local:
```bash
curl -X POST "[https://tu-url-webhook.com](https://tu-url-webhook.com)" \
  -A "Mozilla/5.0" \
  -H "CF-Access-Client-Id: ${{ secrets.CF_CLIENT_ID }}" \
  -H "CF-Access-Client-Secret: ${{ secrets.CF_CLIENT_SECRET }}"
