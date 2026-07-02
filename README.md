Extensión EP3 — Observabilidad y Entornos Reales en DevOps
Autores: Matías Bustos - Roberto González
Asignatura: Ingeniería DevOps — DOY0101
Encargo: Evaluación Parcial N°3

Descripción
Esta sección documenta la extensión del pipeline DevOps desarrollado en evaluaciones anteriores, incorporando mecanismos de observabilidad, métricas de desempeño y validación de cumplimiento normativo sobre la infraestructura desplegada en AWS.
Monitoreo con Amazon CloudWatch
Se habilitó el CloudWatch Agent sobre la instancia EC2 que aloja los contenedores de la aplicación, utilizando AWS Systems Manager (SSM) para su configuración remota sin necesidad de exponer accesos administrativos adicionales (SSH).
El agente recopila:

Métricas de uso: utilización de CPU, uso de memoria, tráfico de red y procesos activos de la instancia.
Logs de aplicación: registros generados por los contenedores Docker (frontend, backend y base de datos), enviados al log group PENDIENTE en CloudWatch Logs.

Para validar el correcto funcionamiento del monitoreo ante cambios de carga real, se generó estrés de CPU y memoria mediante la herramienta stress-ng, confirmando que las métricas se actualizan en tiempo real dentro de CloudWatch.
Dashboard de métricas
Se construyó un dashboard personalizado en CloudWatch (MONITOREO_EV_3_MATIAS_BUSTOS_ROBERTO_GONZALEZ) que centraliza las métricas clave del proyecto:

Uso de CPU y memoria de la instancia.
Errores registrados, obtenidos a partir de los logs de los contenedores.
Métricas asociadas al pipeline CI/CD (tiempo de despliegue y cobertura de pruebas), trazables desde la ejecución de GitHub Actions.

Este dashboard permite tomar decisiones técnicas informadas: por ejemplo, detectar saturación de recursos antes de que afecte la disponibilidad del servicio, o identificar errores recurrentes en los logs sin necesidad de acceder manualmente a la instancia.
Políticas de cumplimiento y detención automática del pipeline
El pipeline CI/CD incorpora controles de calidad y seguridad que se ejecutan de forma automática en cada push a la rama main:

Snyk audita las dependencias del backend y bloquea el pipeline si se detectan vulnerabilidades de severidad alta o crítica.
SonarCloud analiza la calidad del código (code smells, duplicaciones, cobertura) tras superar la etapa de seguridad.
Solo si ambas etapas son exitosas, el pipeline avanza hacia el build de imágenes y el despliegue.

Esto garantiza que, ante una falla crítica de seguridad o calidad, el pipeline se detiene automáticamente y el código no llega al entorno desplegado en producción simulada.
Integración en el flujo CI/CD
La relación entre estas herramientas y el pipeline es la siguiente: el código pasa primero por pruebas automatizadas y validaciones de seguridad/calidad antes de construirse y desplegarse; una vez en producción, el monitoreo con CloudWatch permite observar el comportamiento real del sistema y detectar anomalías que no se evidencian en las etapas previas del pipeline, cerrando así el ciclo de observabilidad de desarrollo a producción.

Uso de Inteligencia Artificial
Durante el desarrollo de esta extensión se utilizó Claude (Anthropic) como herramienta de apoyo para la configuración del agente de CloudWatch, la verificación de logs y métricas, y la redacción de esta documentación. Las decisiones técnicas, la implementación y las validaciones fueron realizadas por los autores.