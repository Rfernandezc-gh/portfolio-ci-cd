# 🏗️ Documentación de Infraestructura: CI/CD Blindado

## 1. Presentación
Este proyecto implementa un pipeline de **Integración y Despliegue Continuo (CI/CD)** automatizado. Su función principal es detectar cualquier cambio en el código fuente, empaquetarlo en una imagen ligera y desplegarlo en un servidor remoto de forma segura y sin intervención manual.

Aunque este flujo está optimizado para **Portainer** sobre **Oracle Cloud**, su lógica basada en contenedores e imágenes es totalmente compatible con entornos de **Kubernetes**, lo que lo hace una solución escalable y profesional.

> **¿Por qué este proyecto?** En lugar de subir archivos por FTP o entrar por SSH, este sistema garantiza que el servidor siempre tenga la versión exacta del código que está en la rama principal, eliminando el error humano.

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

### Estructura del Proyecto
```text
├── .github/workflows/deploy.yml  # Configuración del Pipeline
├── Dockerfile                    # Receta de construcción de la imagen
├── docker-compose.yml            # Definición del servicio en el servidor
└── src/                          # Código fuente de la aplicación
