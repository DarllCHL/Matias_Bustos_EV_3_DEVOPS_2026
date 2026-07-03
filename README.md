





# Extensión EP3 — Observabilidad y Entornos Reales en DevOps

**Autores:** Matías Bustos - Roberto González
**Asignatura:** Ingeniería DevOps — DOY0101
**Encargo:** Evaluación Parcial N°3

---

## Descripción

Esta sección documenta la extensión del pipeline DevOps desarrollado en evaluaciones anteriores, incorporando mecanismos de observabilidad, métricas de desempeño y validación de cumplimiento normativo sobre la infraestructura desplegada en AWS.

## Monitoreo con Amazon CloudWatch

Se habilitó el **CloudWatch Agent** sobre la instancia EC2 que aloja los contenedores de la aplicación, utilizando **AWS Systems Manager (SSM)** para su configuración remota sin necesidad de exponer accesos administrativos adicionales (SSH). La configuración del agente se aplicó mediante un rol IAM asociado a la instancia (`LabInstanceProfile`), lo que permitió el envío de métricas y logs sin exponer credenciales adicionales.

El agente recopila:

- **Métricas de uso**: utilización de CPU (idle, user, system), uso de memoria, tráfico de red (bytes recibidos/enviados) y procesos activos de la instancia.
- **Logs de aplicación**: registros generados por los contenedores Docker (frontend, backend y base de datos), enviados al log group `/tienda-tech/docker-containers` en CloudWatch Logs, usando como log stream el ID de la instancia.

Para validar el correcto funcionamiento del monitoreo ante cambios de carga real, se generó estrés de CPU y memoria mediante la herramienta `stress-ng`, confirmando que las métricas se actualizan en tiempo real dentro de CloudWatch.

### Disponibilidad del servicio

Como mecanismo de verificación de disponibilidad del microservicio, el pipeline de despliegue incorpora un **health check automatizado** ejecutado directamente sobre la instancia EC2 tras cada despliegue. El script consulta el endpoint `http://localhost:3001/api/health` mediante `curl` y valida el código de respuesta HTTP:

- Si la respuesta es `200`, el despliegue se marca como exitoso.
- Si la respuesta es distinta de `200` (incluyendo error de conexión), el pipeline registra los logs del contenedor backend y finaliza con `exit 1`, dejando explícito que el servicio no está disponible.

Este mecanismo permite detectar de forma automática si la aplicación quedó operativa después de cada despliegue, sin intervención manual.

## Dashboard de métricas

Se construyó un dashboard personalizado en CloudWatch (`MONITOREO_EV_3_MATIAS_BUSTOS`) que centraliza las métricas clave del proyecto:

- Uso de CPU y memoria de la instancia (`mem_used`, `cpu_usage_user`, `cpu_usage_system`).
- Errores registrados, obtenidos a partir de los logs de los contenedores.
- **Tiempo de despliegue**, trazable desde la duración total de cada ejecución del pipeline en GitHub Actions (por ejemplo, 3 minutos 46 segundos en la corrida de referencia).
- **Cobertura de pruebas**, obtenida a partir de la ejecución de tests automatizados del backend dentro del pipeline CI/CD.

Este dashboard permite tomar decisiones técnicas informadas: por ejemplo, detectar saturación de recursos antes de que afecte la disponibilidad del servicio, o identificar errores recurrentes en los logs sin necesidad de acceder manualmente a la instancia.

## Políticas de cumplimiento y detención automática del pipeline

El pipeline CI/CD incorpora controles de calidad y seguridad que se ejecutan de forma automática en cada push a las ramas `main` y `pipeline-ci`:

- **Snyk** audita las dependencias del backend (`snyk test --severity-threshold=high`) y bloquea el pipeline si se detectan vulnerabilidades de severidad alta o crítica.
- **SonarCloud** analiza la calidad del código (code smells, duplicaciones, cobertura) tras superar la etapa de seguridad.
- Solo si ambas etapas son exitosas, el pipeline avanza hacia el build de imágenes y el despliegue.

Esto garantiza que, ante una falla crítica de seguridad o calidad, el pipeline se detiene automáticamente y el código no llega al entorno desplegado en producción simulada.

### Evidencia de detención automática ante falla crítica

Para validar este mecanismo, se realizó una prueba controlada introduciendo de forma intencional una versión vulnerable de la dependencia `lodash` (`4.17.4`) en el `package.json` del backend. Al ejecutarse el pipeline, el job **Snyk Security Scan** detectó automáticamente **7 vulnerabilidades de severidad alta** asociadas a esa versión (Arbitrary Code Injection, Code Injection y múltiples casos de Prototype Pollution), documentadas con sus respectivos identificadores en la base de datos de Snyk.

Como resultado:

- El proceso de Snyk finalizó con `exit code 1`, deteniendo el pipeline en esa etapa.
- Los jobs posteriores (**SonarCloud Analysis**, **Build Docker Images**, **Deploy with Docker Compose to AWS EC2**) quedaron marcados como omitidos (*skipped*), es decir, nunca se ejecutaron.
- Como consecuencia, el código con la dependencia vulnerable **no llegó a construirse ni a desplegarse** en el entorno productivo simulado.

Tras confirmar y documentar esta evidencia, la dependencia vulnerable fue revertida a su estado original.

*(Adjuntar aquí la captura del job "Snyk Security Scan" en estado fallido, con el detalle de las vulnerabilidades detectadas.)*

## Integración en el flujo CI/CD

La relación entre estas herramientas y el pipeline es la siguiente: el código pasa primero por pruebas automatizadas y validaciones de seguridad/calidad antes de construirse y desplegarse; una vez en producción, el monitoreo con CloudWatch permite observar el comportamiento real del sistema y detectar anomalías que no se evidencian en las etapas previas del pipeline, cerrando así el ciclo de observabilidad de desarrollo a producción.

## Decisión técnica: despliegue en EC2 con Docker Compose en lugar de Kubernetes

Para el despliegue de los microservicios se optó por una instancia **EC2 con Amazon Linux**, orquestando los contenedores (frontend, backend y base de datos) mediante **Docker Compose**, en lugar de un entorno orquestado como Kubernetes. Esta decisión se tomó considerando el alcance del proyecto y el tiempo disponible para la implementación, priorizando contar con un entorno funcional, estable y con observabilidad real (CloudWatch, health checks, logs centralizados) por sobre la complejidad adicional de administrar un clúster de Kubernetes.

El despliegue se realiza de forma automatizada desde el pipeline de GitHub Actions mediante conexión SSH a la instancia EC2, donde se ejecutan los comandos de `docker compose down`, `docker compose up --build -d` y la verificación posterior mediante el health check descrito anteriormente.

Se deja constancia de que este enfoque no corresponde a un entorno orquestado en el sentido estricto solicitado por la evaluación, siendo esta una limitación conocida del alcance final del proyecto.

---

## Reflexiones individuales

> *Sección de reflexión personal y obligatoria por integrante, redactada sin apoyo de Inteligencia Artificial, según lo indicado en la pauta de evaluación.*

### Matías Bustos

Pa mi este trabajo represento un desafio técnico bastante relevante ya que fue algo con una complejidad que nunca me había enfrentado hasta antes de este curso, debo de reconocer que me costo pero verlo realizado me llena de orgullo, aprendi mucho sobre el uso de github, AWS sobre todo las instancias, la sufri pero lo logré al final, lo que me mantiene satifescho viendo el resultado de lo presentado y estoy listo para el examen, siento personalmente que me voy a defender de manera bastante decente, no sere la herramienta mas afilada del cobertizo pero le pongo empeño

### Roberto González

*(Espacio para reflexión personal: aprendizajes, dificultades enfrentadas y contribución individual al proyecto.)*

---

## Uso de Inteligencia Artificial

Durante el desarrollo de esta extensión se utilizó Claude (Anthropic) como herramienta de apoyo para la configuración del agente de CloudWatch, la verificación de logs y métricas, el diseño y ejecución de la prueba de detención automática del pipeline (IE6), y la redacción de esta documentación. Las decisiones técnicas, la implementación y las validaciones fueron realizadas por los autores.

Referencia de citación: https://bibliotecas.duoc.cl/ia

