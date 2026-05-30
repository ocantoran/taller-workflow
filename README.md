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
