# Manual técnico de taller: Implementación de Workflow Automation en Datadog

## Control del documento

| Campo | Valor |
|---|---|
| Nombre del documento | Manual técnico de taller: Implementación de Workflow Automation en Datadog |
| Versión | 0.1 |
| Estado | Borrador |
| Autor | `<Nombre del autor / equipo>` |
| Fecha | `<AAAA-MM-DD>` |
| Plataforma | Datadog |
| Tipo de documento | Manual técnico de taller |

---

## Índice

- [1. Objetivo](#1-objetivo)
- [2. Alcance](#2-alcance)
- [3. Preparación previa al taller](#3-preparación-previa-al-taller)
- [4. Instalación del Datadog Agent](#4-instalación-del-datadog-agent)
- [5. Validación inicial del Agent](#5-validación-inicial-del-agent)
- [6. Permisos y roles necesarios](#6-permisos-y-roles-necesarios)
- [7. Flujo general de la solución](#7-flujo-general-de-la-solución)
- [8. Monitoreo del servicio](#8-monitoreo-del-servicio)
- [9. Creación del monitor](#9-creación-del-monitor)
- [10. Creación del workflow](#10-creación-del-workflow)
- [11. Asociación del monitor con el workflow](#11-asociación-del-monitor-con-el-workflow)
- [12. Validación de la remediación](#12-validación-de-la-remediación)
- [13. Troubleshooting básico](#13-troubleshooting-básico)
- [14. Referencias oficiales](#14-referencias-oficiales)
- [15. Bitácora de cambios](#15-bitácora-de-cambios)

---

## 1. Objetivo

Documentar el procedimiento para implementar, de forma guiada, un caso práctico de automatización operativa utilizando Datadog Workflow Automation.

Durante el taller se configurará un escenario controlado en el que Datadog detecta un servicio detenido en un host Linux o Windows, genera una alerta mediante un monitor y ejecuta un workflow para validar el evento y coordinar una acción de remediación controlada.

El objetivo es que los participantes comprendan el flujo completo de implementación, desde la preparación del host y la instalación del Datadog Agent, hasta la creación del monitor, la construcción del workflow y la validación del resultado.

> [!NOTE]
> Este manual está enfocado en un taller práctico de laboratorio. No pretende cubrir todos los posibles casos de uso de Workflow Automation ni sustituir un diseño productivo de automatización.

---

## 2. Alcance

Este manual cubre la implementación básica de Workflow Automation en Datadog para un caso práctico de remediación controlada de servicios en servidores Linux o Windows.

El procedimiento contempla:

- Uso de una cuenta de prueba u organización Datadog disponible.
- Instalación básica del Datadog Agent.
- Configuración del monitoreo de un servicio del sistema operativo.
- Creación de un monitor para detectar el servicio detenido.
- Creación de un workflow de remediación controlada.
- Asociación del monitor con el workflow.
- Validación del flujo completo de automatización.
- Revisión básica del resultado de la ejecución.

Este manual **no contempla**:

- Implementaciones productivas avanzadas.
- Diseño avanzado de dashboards, SLOs, logs, trazas o APM.
- Automatizaciones complejas con aprobaciones, múltiples ramas o integraciones externas.
- Administración completa de servicios Linux o Windows.
- Troubleshooting profundo del Datadog Agent o de Workflow Automation.
- Remediación sobre servicios críticos o productivos.

> [!IMPORTANT]
> El taller debe realizarse en un ambiente controlado. No se recomienda ejecutar la primera prueba sobre servidores productivos, servicios críticos o componentes necesarios para el acceso remoto al host.

---

## 3. Preparación previa al taller

Antes del taller, se recomienda completar esta preparación inicial para evitar consumir tiempo durante la sesión práctica. El objetivo es llegar con una cuenta de Datadog activa, una API key disponible y un host listo para instalar o reutilizar el Datadog Agent.

### 3.1 Crear cuenta de Datadog

Para realizar el taller se requiere una cuenta de Datadog. Si aún no se cuenta con una, se debe crear una cuenta de prueba desde el portal oficial de [Datadog Free Trial](https://www.datadoghq.com/free-datadog-trial/).

Pasos generales:

1. Ingresar al portal de prueba de Datadog.
2. Capturar el correo que se utilizará para el taller.
3. Seleccionar **Start your free trial** o **Get Started Free**.
4. Completar el formulario de registro.
5. Seleccionar la región:

   ```text
   United States (US1-East)
   ```

6. Crear la contraseña y completar los datos solicitados.
7. Confirmar el correo electrónico si la plataforma lo solicita.
8. Iniciar sesión y validar acceso a la consola de Datadog.

![Portal de prueba de Datadog](./img/03-01-datadog-free-trial.png)

![Formulario de registro](./img/03-02-formulario-registro.png)

![Selección de región US1-East](./img/03-03-seleccion-region-us1-east.png)

> [!IMPORTANT]
> Para este taller se utilizará la región **United States (US1-East)**. Esta región corresponde al valor `site: datadoghq.com`, que se usará durante la instalación o configuración del Datadog Agent.

> [!WARNING]
> No se recomienda utilizar correos temporales. Si se pierde acceso al correo utilizado, puede ser más difícil recuperar la cuenta o continuar con actividades posteriores.

---

### 3.2 Obtener API key

La API key será necesaria para instalar o configurar el Datadog Agent. Esta llave permite que el Agent envíe información desde el host hacia la cuenta de Datadog utilizada en el taller.

Pasos generales:

1. Iniciar sesión en Datadog.
2. Ir a **Organization Settings**.
3. Entrar a **API Keys**.
4. Crear una nueva API key o utilizar una API key existente autorizada.
5. Guardar la API key de forma segura.

La gestión de llaves puede consultarse en la documentación de [API and Application Keys](https://docs.datadoghq.com/account_management/api-app-keys/).

![Sección API Keys](./img/03-04-api-keys.png)

Datos que deben conservarse para el taller:

| Dato | Valor |
|---|---|
| Correo utilizado | `<correo_del_taller>` |
| Región seleccionada | `United States (US1-East)` |
| Site del Agent | `datadoghq.com` |
| API key | `<API_KEY_DEL_TALLER>` |

> [!IMPORTANT]
> La API key debe tratarse como información sensible. No debe compartirse en repositorios públicos, capturas de pantalla, chats abiertos o documentación sin protección.

---

### 3.3 Preparar host para el taller

Se requiere un host Linux o Windows para instalar o reutilizar el Datadog Agent.

Primero se debe validar si el Agent ya está instalado.

#### Linux

```bash
sudo systemctl status datadog-agent
sudo datadog-agent status
```

#### Windows PowerShell

```powershell
Get-Service datadogagent
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" status
```

Si el Agent no está instalado, se instalará en la sección [4. Instalación del Datadog Agent](#4-instalación-del-datadog-agent).

Si el Agent ya está instalado, se debe validar que apunte al site y API key del taller.

Valor esperado:

```yaml
site: datadoghq.com
api_key: <API_KEY_DEL_TALLER>
```

Archivos de configuración comunes:

| Sistema operativo | Archivo |
|---|---|
| Linux | `/etc/datadog-agent/datadog.yaml` |
| Windows | `C:\ProgramData\Datadog\datadog.yaml` |

Después de modificar el archivo, se debe reiniciar el Agent.

#### Linux

```bash
sudo systemctl restart datadog-agent
sudo systemctl status datadog-agent
```

#### Windows PowerShell

```powershell
Restart-Service datadogagent
Get-Service datadogagent
```

![Validación del Agent](./img/03-05-validacion-agent.png)

> [!WARNING]
> Cambiar la API key de un Agent existente puede hacer que el host deje de reportar a otra cuenta de Datadog y comience a reportar a la cuenta utilizada en el taller. Antes de realizar este cambio, se debe confirmar que está autorizado.

---

### 3.4 Seleccionar servicio de prueba

Para validar el monitoreo y la remediación, se debe elegir un servicio de bajo impacto. El objetivo es simular un servicio detenido sin afectar una aplicación crítica o un componente esencial del sistema operativo.

Servicios sugeridos:

| Sistema operativo | Servicios sugeridos |
|---|---|
| Linux | `cups.service`, `atd.service`, `avahi-daemon.service` |
| Windows | `Spooler`, `WSearch`, `Fax` |

Antes de seleccionarlo, se debe validar que el servicio exista y que pueda detenerse e iniciarse sin impacto relevante.

#### Linux

```bash
systemctl status cups.service
systemctl status atd.service
systemctl status avahi-daemon.service
```

#### Windows PowerShell

```powershell
Get-Service -Name Spooler
Get-Service -Name WSearch
Get-Service -Name Fax
```

> [!CAUTION]
> No se deben utilizar servicios críticos para la prueba inicial.

---

## 4. Instalación del Datadog Agent

En esta sección se instalará el Datadog Agent en el host de laboratorio. Si el Agent ya se encuentra instalado y configurado con la API key del taller, se puede continuar con la sección [5. Validación inicial del Agent](#5-validación-inicial-del-agent).

Para este taller se utilizarán los siguientes valores:

```yaml
site: datadoghq.com
api_key: <API_KEY_DEL_TALLER>
```

Selecciona la ruta correspondiente al sistema operativo del host de laboratorio:

| Sistema operativo | Ruta de instalación |
|---|---|
| Linux | [Ir a instalación en Linux](#41-instalación-en-linux) |
| Windows | [Ir a instalación en Windows](#42-instalación-en-windows) |

> [!IMPORTANT]
> La API key real no debe guardarse en el repositorio del taller. En la documentación se debe utilizar siempre un placeholder como `<API_KEY_DEL_TALLER>`.

---

### 4.1 Instalación en Linux

Para instalar el Agent en Linux, se puede utilizar la guía oficial de [Datadog Agent para Linux](https://docs.datadoghq.com/agent/supported_platforms/linux/).

Pasos generales:

1. Iniciar sesión en Datadog.
2. Ir a la sección de instalación del Agent.
3. Seleccionar **Linux** como plataforma.
4. Validar que la API key corresponde al taller.
5. Validar que el site sea:

   ```text
   datadoghq.com
   ```

6. Copiar el comando generado por Datadog.
7. Ejecutar el comando en el host Linux con permisos administrativos.

![Instalación del Agent en Linux](./img/04-01-agent-linux-install.png)

Ejemplo de comando de referencia:

```bash
DD_API_KEY="<API_KEY_DEL_TALLER>" DD_SITE="datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

Al finalizar la instalación, validar que el servicio esté activo:

```bash
sudo systemctl status datadog-agent
```

También se puede validar el estado general del Agent con:

```bash
sudo datadog-agent status
```

---

### 4.2 Instalación en Windows

Para instalar el Agent en Windows, se puede utilizar la guía oficial de [Datadog Agent para Windows](https://docs.datadoghq.com/agent/supported_platforms/windows/).

Pasos generales:

1. Iniciar sesión en Datadog.
2. Ir a la sección de instalación del Agent.
3. Seleccionar **Windows** como plataforma.
4. Descargar el instalador del Datadog Agent.
5. Ejecutar el instalador como administrador.
6. Ingresar la API key del taller cuando el instalador lo solicite.
7. Validar que el site sea:

   ```text
   datadoghq.com
   ```

8. Finalizar la instalación.

![Instalación del Agent en Windows](./img/04-02-agent-windows-install.png)

Al finalizar la instalación, validar que el servicio exista:

```powershell
Get-Service datadogagent
```

También se puede validar el estado general del Agent con:

```powershell
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" status
```

---

### 4.3 Resultado esperado

Al finalizar esta sección, el host debe contar con el Datadog Agent instalado y ejecutándose.

El Agent debe estar configurado con:

```yaml
site: datadoghq.com
api_key: <API_KEY_DEL_TALLER>
```

En la siguiente sección se validará que el host reporte correctamente hacia Datadog.

---

## 5. Validación inicial del Agent

En esta sección se validará que el Datadog Agent esté instalado, en ejecución y reportando información hacia Datadog.

Selecciona la ruta correspondiente:

| Sistema operativo | Ruta de validación |
|---|---|
| Linux | [Ir a validación en Linux](#51-validación-en-linux) |
| Windows | [Ir a validación en Windows](#52-validación-en-windows) |
| Datadog | [Ir a validación en Datadog](#53-validación-en-datadog) |

---

### 5.1 Validación en Linux

Ejecutar:

```bash
sudo systemctl status datadog-agent
sudo datadog-agent status
```

Resultado esperado:

- El servicio `datadog-agent` debe estar activo.
- El comando `datadog-agent status` debe mostrar información del Agent.
- No deben aparecer errores críticos de conexión o autenticación.

![Validación del Agent en Linux](./img/05-01-validacion-agent-linux.png)

---

### 5.2 Validación en Windows

Ejecutar:

```powershell
Get-Service datadogagent
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" status
```

Resultado esperado:

- El servicio `datadogagent` debe estar en estado `Running`.
- El comando `agent.exe status` debe mostrar información del Agent.
- No deben aparecer errores críticos de conexión o autenticación.

![Validación del Agent en Windows](./img/05-02-validacion-agent-windows.png)

---

### 5.3 Validación en Datadog

Desde la consola de Datadog:

1. Ir a **Infrastructure**.
2. Entrar a **Host List** o **Host Map**.
3. Buscar el hostname del servidor de laboratorio.
4. Confirmar que el host aparece reportando información.

![Validación del host en Datadog](./img/05-03-validacion-host-datadog.png)

> [!NOTE]
> El host puede tardar algunos minutos en aparecer después de instalar o reiniciar el Agent.

---

### 5.4 Resultado esperado

Al finalizar esta sección, se debe confirmar que:

| Validación | Resultado esperado |
|---|---|
| Servicio del Agent | Activo o en ejecución. |
| Estado del Agent | Sin errores críticos. |
| Host en Datadog | Visible en Infrastructure. |
| Site configurado | `datadoghq.com`. |

Si el Agent no inicia o el host no aparece en Datadog, revisar la sección [13. Troubleshooting básico](#13-troubleshooting-básico).

---

## 6. Permisos y roles necesarios

Antes de crear el monitor y el workflow, se debe validar que el usuario tenga permisos suficientes dentro de Datadog.

Si la cuenta fue creada para el taller, normalmente el usuario tendrá permisos suficientes para continuar. Si se utiliza una cuenta compartida o una organización existente, se deben validar los permisos asignados mediante [RBAC](https://docs.datadoghq.com/account_management/rbac/).

### 6.1 Validaciones mínimas

El usuario debe poder acceder o realizar las siguientes acciones:

| Elemento | Validación |
|---|---|
| **Infrastructure** | Consultar el host que reporta desde el Datadog Agent. |
| **Integrations** | Consultar o modificar configuraciones necesarias del Agent o integraciones. |
| **Monitors** | Crear o editar el monitor que detectará el servicio detenido. |
| **Workflow Automation** | Crear, editar y ejecutar workflows. |
| **Connections / credentials** | Configurar conexiones o credenciales si el workflow lo requiere. |

![Validación de permisos en Datadog](./img/06-01-validacion-permisos-datadog.png)

### 6.2 Validación rápida en la interfaz

Desde Datadog, validar que se puede acceder a:

1. **Infrastructure**.
2. **Monitors**.
3. **Workflow Automation**.
4. **Organization Settings**, si se requiere revisar roles o API keys.

![Acceso a secciones necesarias](./img/06-02-secciones-datadog.png)

### 6.3 Consideraciones

Si el usuario no puede crear monitores o workflows, se debe solicitar el ajuste de permisos correspondiente antes de continuar.

Para ambientes con roles personalizados, se puede consultar la referencia de [Datadog Role Permissions](https://docs.datadoghq.com/account_management/rbac/permissions/).

---

## 7. Flujo general de la solución

El flujo general del taller consiste en detectar un servicio detenido desde Datadog, generar una alerta mediante un monitor y ejecutar un workflow que coordine la validación y la acción de remediación controlada.

La solución se compone de los siguientes elementos:

| Componente | Función |
|---|---|
| **Datadog Agent** | Reporta información del host y del servicio monitoreado. |
| **Service Check / Monitoreo del servicio** | Valida si el servicio está activo o detenido. |
| **Monitor** | Genera una alerta cuando el servicio cumple la condición definida. |
| **Workflow Automation** | Recibe el trigger del monitor y ejecuta las actions configuradas. |
| **Sistema de ejecución autorizado** | Ejecuta la acción controlada sobre el host, cuando aplique. |
| **Notificación / Registro** | Informa el resultado de la ejecución o registra evidencia del proceso. |

### 7.1 Flujo operativo

```text
Datadog Agent reporta estado del servicio
  ↓
Datadog evalúa el estado mediante monitoreo
  ↓
Monitor detecta servicio detenido
  ↓
Monitor ejecuta el trigger del workflow
  ↓
Workflow valida el contexto recibido
  ↓
Workflow coordina la acción de remediación controlada
  ↓
Se valida el resultado
  ↓
Se notifica o registra la ejecución
```

![Flujo general de la solución](./img/07-01-flujo-general-solucion.png)

### 7.2 Resultado esperado del flujo

Al finalizar el flujo, se debe poder comprobar que:

- El servicio detenido fue detectado por Datadog.
- El monitor generó la alerta correspondiente.
- El workflow se ejecutó a partir del trigger del monitor.
- La acción de remediación o validación fue ejecutada de forma controlada.
- El resultado quedó visible en la workflow execution o en la notificación configurada.

> [!IMPORTANT]
> Workflow Automation coordina el flujo de automatización. Si se requiere ejecutar una acción directamente sobre un servidor, debe existir un mecanismo autorizado para hacerlo, como una integración, connection, webhook, API o plataforma externa de automatización.

---

## 8. Monitoreo del servicio

En esta sección se configurará el monitoreo del servicio seleccionado para el taller.  
La configuración depende del sistema operativo del host de laboratorio.

Selecciona la ruta correspondiente:

| Sistema operativo | Ruta de configuración |
|---|---|
| Linux | [Ir a monitoreo en Linux](#81-monitoreo-en-linux) |
| Windows | [Ir a monitoreo en Windows](#82-monitoreo-en-windows) |

---

### 8.1 Monitoreo en Linux

Para Linux se utilizará la integración de [Systemd](https://docs.datadoghq.com/integrations/systemd/), la cual permite monitorear unidades administradas por `systemd`.

Editar el archivo de configuración:

```bash
sudo vi /etc/datadog-agent/conf.d/systemd.d/conf.yaml
```

Agregar el servicio seleccionado:

```yaml
init_config:

instances:
  - unit_names:
      - cups.service
```

> [!NOTE]
> Reemplazar `cups.service` por el servicio seleccionado para el taller, por ejemplo `atd.service` o `avahi-daemon.service`.

Reiniciar el Agent:

```bash
sudo systemctl restart datadog-agent
```

Validar el check:

```bash
sudo -u dd-agent datadog-agent check systemd
```

![Configuración de Systemd en Linux](./img/08-01-systemd-linux.png)

---

### 8.2 Monitoreo en Windows

Para Windows se utilizará la integración de [Windows Services](https://docs.datadoghq.com/integrations/windows-service/), la cual permite monitorear el estado de servicios de Windows.

Editar el archivo de configuración:

```powershell
notepad "C:\ProgramData\Datadog\conf.d\windows_service.d\conf.yaml"
```

Agregar el servicio seleccionado:

```yaml
init_config:

instances:
  - services:
      - spooler
```

> [!NOTE]
> Reemplazar `spooler` por el servicio seleccionado para el taller, por ejemplo `wsearch` o `fax`. Se debe utilizar el **service name**, no el display name.

Reiniciar el Agent:

```powershell
Restart-Service datadogagent
```

Validar el check:

```powershell
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" check windows_service
```

![Configuración de Windows Service](./img/08-02-windows-service.png)

---

### 8.3 Resultado esperado

Al finalizar esta sección, Datadog debe recibir el estado del servicio monitoreado.

| Sistema operativo | Check esperado |
|---|---|
| Linux | `systemd` |
| Windows | `windows_service` |

En la siguiente sección se creará el monitor que utilizará este estado para detectar cuando el servicio se encuentre detenido.

---

## 9. Creación del monitor

En esta sección se creará el monitor que detectará cuando el servicio seleccionado se encuentre detenido.

Para este taller se utilizará un monitor de tipo **Service Check**, ya que el estado del servicio será reportado por el Datadog Agent mediante la integración configurada en la sección anterior.

Selecciona el check correspondiente según el sistema operativo:

| Sistema operativo | Service Check |
|---|---|
| Linux | `systemd.unit.state` |
| Windows | `windows_service.state` |

---

### 9.1 Crear monitor de Service Check

Desde la consola de Datadog:

1. Ir a **Monitors**.
2. Seleccionar **New Monitor**.
3. Seleccionar **Service Check**.
4. Elegir el service check correspondiente:
   - Linux: `systemd.unit.state`
   - Windows: `windows_service.state`
5. Definir el scope del monitor usando el host y el servicio seleccionado.
6. Configurar la alerta para dispararse cuando el check reporte estado `CRITICAL`.
7. Configurar la recuperación cuando el check vuelva a estado `OK`.
8. Asignar nombre y mensaje al monitor.
9. Guardar el monitor.

![Creación del Service Check Monitor](./img/09-01-service-check-monitor.png)

---

### 9.2 Configuración sugerida

| Campo | Valor sugerido |
|---|---|
| Tipo de monitor | `Service Check` |
| Check Linux | `systemd.unit.state` |
| Check Windows | `windows_service.state` |
| Condición de alerta | `CRITICAL` |
| Recuperación | `OK` |
| Scope | Host y servicio de laboratorio |
| Notificación | La requerida para el taller |

> [!NOTE]
> El scope exacto puede variar según los tags que genere el check. Durante el taller se debe validar en la interfaz que el monitor esté apuntando al host y servicio correctos.

---

### 9.3 Nombre sugerido del monitor

```text
[Workshop] Servicio detenido detectado - <HOST> - <SERVICIO>
```

Ejemplo:

```text
[Workshop] Servicio detenido detectado - lab-linux-01 - cups.service
```

---

### 9.4 Mensaje sugerido del monitor

```text
Se detectó que el servicio monitoreado se encuentra detenido.

Host: {{host.name}}
Estado del monitor: {{value}}

Este monitor forma parte del taller de Datadog Workflow Automation.
```

> [!IMPORTANT]
> En este punto el monitor solo detecta la condición de falla. La asociación con el workflow se realizará en la sección [11. Asociación del monitor con el workflow](#11-asociación-del-monitor-con-el-workflow).

---

## 10. Creación del workflow

En esta sección se creará el workflow que será ejecutado cuando el monitor detecte el servicio detenido.

El workflow recibirá el contexto del monitor, validará la información básica del evento y ejecutará una action de remediación controlada mediante un mecanismo autorizado.

> [!IMPORTANT]
> Workflow Automation coordina la automatización, pero no debe entenderse como un agente remoto para ejecutar comandos directamente sobre el servidor. Para iniciar un servicio en el host se requiere una integración, webhook, API o plataforma externa autorizada.

### 10.1 Crear workflow

Desde la consola de Datadog:

1. Ir a **Workflow Automation**.
2. Seleccionar **New Workflow**.
3. Asignar un nombre al workflow.

Nombre sugerido:

```text
[Workshop] Remediación de servicio detenido
```

![Creación del workflow](./img/10-01-crear-workflow.png)

---

### 10.2 Agregar trigger del monitor

Agregar un trigger de tipo **Monitor** para que el workflow pueda ejecutarse desde el monitor creado en la sección anterior.

Pasos generales:

1. Agregar un nuevo trigger.
2. Seleccionar **Monitor Trigger**.
3. Guardar el workflow.
4. Validar el nombre o handle del workflow, ya que se utilizará al asociarlo con el monitor.

![Monitor Trigger en Workflow Automation](./img/10-02-monitor-trigger.png)

> [!NOTE]
> La asociación final entre el monitor y el workflow se realizará en la sección [11. Asociación del monitor con el workflow](#11-asociación-del-monitor-con-el-workflow).

---

### 10.3 Definir información mínima del evento

El workflow debe recibir información suficiente para saber qué servicio atender y en qué host ocurrió la alerta.

Datos mínimos esperados:

| Dato | Descripción |
|---|---|
| `host` | Host donde se detectó el servicio detenido. |
| `service` | Servicio monitoreado. |
| `status` | Estado reportado por el monitor. |
| `message` | Mensaje o contexto del evento. |

![Variables del workflow](./img/10-03-variables-workflow.png)

---

### 10.4 Agregar validación básica

Agregar una condición para validar que el evento contiene los datos mínimos necesarios.

Validación sugerida:

```text
¿El evento contiene host y servicio?
```

Resultado esperado:

| Condición | Acción |
|---|---|
| Sí | Continuar con la remediación controlada. |
| No | Registrar error y finalizar sin ejecutar remediación. |

![Condición de validación](./img/10-04-validacion-contexto.png)

---

### 10.5 Agregar action de remediación controlada

Agregar la action que enviará la solicitud de remediación al sistema autorizado.

Para el taller puede utilizarse una **HTTP action** hacia un webhook o API interna.

Ejemplo conceptual:

```text
POST <URL_WEBHOOK_REMEDIACION>
```

Body de referencia:

```json
{
  "host": "<HOST>",
  "service": "<SERVICIO>",
  "action": "start_service",
  "source": "datadog_workflow_automation"
}
```

![HTTP action de remediación](./img/10-05-http-action-remediacion.png)

> [!WARNING]
> La URL, tokens o credenciales utilizadas por la action no deben guardarse directamente en el repositorio. Se deben utilizar connections, credenciales protegidas o placeholders como `<URL_WEBHOOK_REMEDIACION>`.

---

### 10.6 Registrar o notificar resultado

Agregar una action final para registrar o notificar el resultado de la ejecución.

Ejemplos:

- Notificar recuperación exitosa.
- Notificar fallo de remediación.
- Registrar evidencia de la ejecución.
- Escalar el evento si la remediación no fue exitosa.

![Resultado del workflow](./img/10-06-resultado-workflow.png)

---

### 10.7 Resultado esperado

Al finalizar esta sección, se debe contar con un workflow que tenga:

| Elemento | Estado esperado |
|---|---|
| Nombre del workflow | Definido. |
| Monitor trigger | Configurado. |
| Validación básica | Configurada. |
| Action de remediación | Configurada o preparada con placeholder. |
| Notificación o registro | Configurado. |

El workflow quedará listo para asociarse al monitor en la siguiente sección.

---

## 11. Asociación del monitor con el workflow

En esta sección se asociará el monitor creado previamente con el workflow de remediación.

Antes de continuar, validar que:

- El monitor ya existe.
- El workflow ya tiene un **Monitor trigger**.
- El workflow fue guardado y publicado.
- El workflow cuenta con las variables o inputs necesarios para recibir información del monitor.

### 11.1 Agregar workflow al monitor

Desde la consola de Datadog:

1. Ir a **Monitors**.
2. Editar el monitor creado en la sección [9. Creación del monitor](#9-creación-del-monitor).
3. Ir a **Configure notifications & automations**.
4. Seleccionar **Add Workflow**.
5. Buscar y seleccionar el workflow creado en la sección [10. Creación del workflow](#10-creación-del-workflow).
6. Configurar los inputs requeridos, si aplica.
7. Guardar el monitor.

![Asociación del monitor con el workflow](./img/11-01-asociar-monitor-workflow.png)

### 11.2 Parámetros sugeridos

Si el workflow requiere datos del monitor, se pueden pasar variables como referencia.

Ejemplo conceptual:

```text
host={{host.name}}
service=<SERVICIO_DEL_TALLER>
status={{value}}
```

> [!NOTE]
> Las variables exactas disponibles pueden variar según el tipo de monitor. Durante el taller se deben validar desde la opción de variables disponibles en la configuración del monitor.

### 11.3 Resultado esperado

Al finalizar esta sección:

- El monitor debe tener asociado el workflow.
- El workflow debe ejecutarse cuando el monitor entre en condición de alerta.
- El monitor debe conservar su mensaje de alerta y, adicionalmente, la referencia al workflow.

> [!IMPORTANT]
> Cada vez que el monitor dispare el workflow publicado, se generará una workflow execution. Esto debe considerarse para evitar ejecuciones innecesarias durante las pruebas.

---

## 12. Validación de la remediación

En esta sección se validará el flujo completo: detener el servicio de prueba, esperar la alerta del monitor, confirmar la ejecución del workflow y revisar el resultado de la remediación controlada.

> [!IMPORTANT]
> La prueba debe realizarse únicamente sobre el servicio de bajo impacto seleccionado para el taller.

### 12.1 Detener el servicio de prueba

Selecciona la ruta correspondiente:

| Sistema operativo | Ruta de validación |
|---|---|
| Linux | [Ir a prueba en Linux](#1211-prueba-en-linux) |
| Windows | [Ir a prueba en Windows](#1212-prueba-en-windows) |

#### 12.1.1 Prueba en Linux

Detener el servicio seleccionado:

```bash
sudo systemctl stop cups.service
```

Validar que el servicio quedó detenido:

```bash
systemctl status cups.service
```

> [!NOTE]
> Reemplazar `cups.service` por el servicio seleccionado para el taller.

![Servicio detenido en Linux](./img/12-01-servicio-detenido-linux.png)

#### 12.1.2 Prueba en Windows

Detener el servicio seleccionado:

```powershell
Stop-Service -Name Spooler
```

Validar que el servicio quedó detenido:

```powershell
Get-Service -Name Spooler
```

> [!NOTE]
> Reemplazar `Spooler` por el servicio seleccionado para el taller.

![Servicio detenido en Windows](./img/12-02-servicio-detenido-windows.png)

---

### 12.2 Validar alerta del monitor

Después de detener el servicio, esperar a que Datadog reciba el nuevo estado y el monitor cambie a condición de alerta.

Desde Datadog:

1. Ir a **Monitors**.
2. Abrir el monitor creado para el taller.
3. Validar que el monitor cambió a estado de alerta.
4. Confirmar que el host y servicio corresponden a la prueba.

![Monitor en alerta](./img/12-03-monitor-alerta.png)

> [!NOTE]
> El cambio de estado puede tardar algunos minutos, dependiendo del intervalo de recolección del Agent y de la configuración del monitor.

---

### 12.3 Validar ejecución del workflow

Una vez que el monitor entra en alerta, se debe validar que el workflow asociado se ejecutó.

Desde Datadog:

1. Ir a **Workflow Automation**.
2. Abrir el workflow creado para el taller.
3. Revisar el historial de ejecuciones.
4. Abrir la última **workflow execution**.
5. Validar si las actions se ejecutaron correctamente.

![Workflow execution](./img/12-04-workflow-execution.png)

Resultado esperado:

| Validación | Resultado esperado |
|---|---|
| Trigger del monitor | Ejecutó el workflow. |
| Datos recibidos | Incluyen host, servicio y estado del evento. |
| Validación básica | Finalizó correctamente. |
| Action de remediación | Se ejecutó mediante el mecanismo autorizado. |
| Resultado final | Quedó registrado en la workflow execution. |

---

### 12.4 Validar estado final del servicio

Después de la ejecución del workflow, validar si el servicio fue iniciado nuevamente.

#### Linux

```bash
systemctl status cups.service
```

#### Windows PowerShell

```powershell
Get-Service -Name Spooler
```

![Servicio recuperado](./img/12-05-servicio-recuperado.png)

---

### 12.5 Resultado esperado

Al finalizar la validación, se debe comprobar que:

- El servicio detenido fue detectado por Datadog.
- El monitor cambió a estado de alerta.
- El workflow asociado fue ejecutado.
- La remediación controlada fue invocada.
- El resultado quedó registrado en la workflow execution.
- El servicio quedó activo nuevamente o el evento fue escalado correctamente.

> [!WARNING]
> Si el servicio no se recupera, no se debe repetir la prueba de forma indefinida. Primero se debe revisar el resultado de la workflow execution, los logs del Agent y el mecanismo autorizado utilizado para ejecutar la remediación.

---

## 13. Troubleshooting básico

Esta sección contiene validaciones básicas para identificar problemas comunes durante el taller.

### 13.1 El Agent no inicia

Validar el estado del servicio.

#### Linux

```bash
sudo systemctl status datadog-agent
sudo journalctl -u datadog-agent -n 50 --no-pager
```

#### Windows PowerShell

```powershell
Get-Service datadogagent
```

Si el servicio no inicia, revisar que el archivo `datadog.yaml` tenga los valores correctos:

```yaml
api_key: <API_KEY_DEL_TALLER>
site: datadoghq.com
```

![Troubleshooting del Agent](./img/13-01-agent-no-inicia.png)

---

### 13.2 El host no aparece en Datadog

Validar que el Agent esté en ejecución y que no existan errores de conexión o autenticación.

#### Linux

```bash
sudo datadog-agent status
```

#### Windows PowerShell

```powershell
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" status
```

Puntos a revisar:

- API key correcta.
- Site configurado como `datadoghq.com`.
- Conectividad del host hacia Datadog.
- Firewall, proxy o restricciones de red.
- Tiempo de espera después de instalar o reiniciar el Agent.

![Host no visible en Datadog](./img/13-02-host-no-visible.png)

---

### 13.3 El servicio no reporta estado

Validar que la integración del servicio esté configurada correctamente.

#### Linux

```bash
sudo -u dd-agent datadog-agent check systemd
```

#### Windows PowerShell

```powershell
& "C:\Program Files\Datadog\Datadog Agent\bin\agent.exe" check windows_service
```

Puntos a revisar:

- El servicio existe en el sistema operativo.
- El nombre del servicio está escrito correctamente.
- El archivo `conf.yaml` tiene sintaxis válida.
- El Agent fue reiniciado después del cambio.

![Check del servicio](./img/13-03-check-servicio.png)

---

### 13.4 El monitor no cambia a alerta

Si el servicio ya fue detenido pero el monitor no cambia de estado, revisar:

- Que el check correcto esté asociado al monitor.
- Que el scope apunte al host y servicio correctos.
- Que el Agent esté enviando datos recientes.
- Que haya pasado el tiempo suficiente para que Datadog evalúe la condición.

![Monitor sin alerta](./img/13-04-monitor-sin-alerta.png)



### 13.5 El workflow no se ejecuta

Si el monitor entra en alerta pero el workflow no se ejecuta, validar:

- Que el workflow esté publicado.
- Que el workflow tenga configurado un **Monitor trigger**.
- Que el monitor tenga asociado el workflow en **Configure notifications & automations**.
- Que los inputs requeridos por el workflow estén configurados.

![Workflow no ejecutado](./img/13-05-workflow-no-ejecutado.png)


### 13.6 La remediación no funciona

Si el workflow se ejecuta pero el servicio no se inicia nuevamente, revisar:

- El resultado de la **workflow execution**.
- La respuesta de la action de remediación.
- Las credenciales o connection utilizadas.
- El mecanismo externo usado para ejecutar la acción.
- Los permisos sobre el host o servicio objetivo.

> [!IMPORTANT]
> Workflow Automation coordina la acción, pero la ejecución sobre el servidor depende del mecanismo autorizado utilizado, como webhook, API, integración o plataforma externa de automatización.

![Fallo de remediación](./img/13-06-fallo-remediacion.png)

---

## 14. Referencias oficiales

En esta sección se concentran las referencias oficiales utilizadas durante el taller.

| Tema | Referencia |
|---|---|
| Cuenta de prueba | [Datadog Free Trial](https://www.datadoghq.com/free-datadog-trial/) |
| Primeros pasos en Datadog | [Getting Started with Datadog](https://docs.datadoghq.com/getting_started/) |
| Sitios de Datadog | [Datadog Sites](https://docs.datadoghq.com/getting_started/site/) |
| API keys y Application keys | [API and Application Keys](https://docs.datadoghq.com/account_management/api-app-keys/) |
| Instalación del Agent en Linux | [Datadog Agent para Linux](https://docs.datadoghq.com/agent/supported_platforms/linux/) |
| Instalación del Agent en Windows | [Datadog Agent para Windows](https://docs.datadoghq.com/agent/supported_platforms/windows/) |
| Comandos del Agent | [Agent Commands](https://docs.datadoghq.com/agent/configuration/agent-commands/) |
| Archivos de configuración del Agent | [Agent Configuration Files](https://docs.datadoghq.com/agent/configuration/agent-configuration-files/) |
| Integración Systemd | [Systemd Integration](https://docs.datadoghq.com/integrations/systemd/) |
| Integración Windows Service | [Windows Service Integration](https://docs.datadoghq.com/integrations/windows-service/) |
| Monitores en Datadog | [Monitors](https://docs.datadoghq.com/monitors/) |
| Service Check Monitor | [Service Check Monitor](https://docs.datadoghq.com/monitors/types/service_check/) |
| Workflow Automation | [Datadog Workflow Automation](https://docs.datadoghq.com/actions/workflows/) |
| Triggers de workflows | [Trigger a Workflow](https://docs.datadoghq.com/actions/workflows/trigger/) |
| Actions en workflows | [Workflow Actions](https://docs.datadoghq.com/actions/workflows/actions/) |
| Connections | [Connections](https://docs.datadoghq.com/actions/connections/) |
| RBAC | [Role Based Access Control](https://docs.datadoghq.com/account_management/rbac/) |
| Permisos de roles | [Datadog Role Permissions](https://docs.datadoghq.com/account_management/rbac/permissions/) |
```‍```

---

## 15. Bitácora de cambios

| Versión | Fecha | Descripción del cambio | Autor |
|---|---|---|---|
| 0.1 | `<AAAA-MM-DD>` | Creación inicial del manual técnico de taller. | `<Nombre del autor / equipo>` |

