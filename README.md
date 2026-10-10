<div align="center">

# Sothe Windows Optimizer

**El optimizador de Windows para usuarios expertos: potente, sin trucos y 100 % reversible.**

[![Última versión](https://img.shields.io/github/v/release/ImSothe/Sothe-Windows-Optimizer?label=versi%C3%B3n&color=8b5cf6)](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest)
[![Descargas](https://img.shields.io/github/downloads/ImSothe/Sothe-Windows-Optimizer/total?color=22d3ee)](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases)
![Windows 10 | 11](https://img.shields.io/badge/Windows-10%20%7C%2011%20x64-0078D4?logo=windows&logoColor=white)
![.NET 8](https://img.shields.io/badge/.NET-8%20(incluido)-512BD4?logo=dotnet&logoColor=white)
![WPF](https://img.shields.io/badge/UI-WPF-5C2D91)
![Tests](https://img.shields.io/badge/tests-212%20pasan-success)
![Actualizaciones firmadas](https://img.shields.io/badge/actualizaciones-firmadas%20ECDSA%20P--256-8b5cf6)
![Licencia](https://img.shields.io/badge/licencia-freeware%20(c%C3%B3digo%20cerrado)-lightgrey)

[**⬇ Descargar la última versión**](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest) · [Cómo funciona](#arquitectura) · [Seguridad](#seguridad) · [English](#english)

</div>

---

## Qué es

Sothe Optimizer aplica **más de 70 ajustes** de rendimiento, latencia, privacidad y limpieza de Windows pensados para **gaming competitivo** (CS2, Valorant, Overwatch…) y para **laboratorios de ciberseguridad** con máquinas virtuales.

A diferencia de otras herramientas, **no esconde las opciones peligrosas**: deja tocar incluso Defender, el firewall, las mitigaciones Spectre/Meltdown, el hipervisor o UAC. La condición es que **todo se pueda deshacer**: antes de cambiar nada guarda el valor anterior, y un botón de pánico («Revertir TODO») lo restaura.

> **Honestidad ante todo.** En un shooter lo que de verdad marca la diferencia es un monitor de alta frecuencia, Reflex, un *frametime* estable, red por cable y práctica. Los ajustes de registro aportan, con suerte, un dígito porcentual. Esta herramienta te ahorra horas de configuración, no hace milagros.

### Contenido

| | |
|---|---|
| **Ajustes** | 70 en 13 categorías (Rendimiento, Input, Red, GPU, Energía, Privacidad, Interfaz, Almacenamiento, Seguridad, Debloat, Windows Update…) |
| **Niveles de riesgo** | `safe` 🟢 · `moderate` 🔵 · `high` 🟠 · `extreme` 🔴 (**PROHIBIDO**, oculto tras el Modo Avanzado) |
| **Apps (winget)** | 31 programas instalables/desinstalables en lote |
| **Características de Windows** | 6 (Hyper-V, .NET 3.5, WSL…) |
| **Perfiles** | Competitivo · Ciberseguridad · Limpio · los tuyos, con código para compartir (`SOTHE-1-…`) |
| **Reparaciones** | SFC, DISM, reset de red, Windows Update, CHKDSK |

---

## Descarga e instalación

Este repositorio **solo distribuye el ejecutable**. No hay instalador ni dependencias: es **un único `.exe` autocontenido**.

1. Ve a [**Releases**](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest) y descarga **`SotheOptimizer.exe`**.
2. *(Recomendado)* Comprueba su integridad. En PowerShell:

   ```powershell
   Get-FileHash .\SotheOptimizer.exe -Algorithm SHA256
   ```

   El resultado debe coincidir con el campo `sha256` del archivo `release.json` de la misma release.
3. Ejecútalo con doble clic. Windows pedirá **permisos de administrador** (UAC): acepta.
4. Si aparece SmartScreen («Windows protegió su PC»): **Más información → Ejecutar de todas formas**. El ejecutable aún no está firmado con un certificado de pago (cuesta ~150–400 €/año); por eso la integridad se garantiza con las firmas y hashes descritos más abajo.
5. Lee y acepta el aviso legal, **crea un punto de restauración** desde la propia app y empieza por un perfil.

### Requisitos

- **Windows 10 o 11, 64 bits.**
- **Permisos de administrador**: la herramienta modifica el registro (HKLM), servicios y características del sistema. Sin elevación Windows no deja ejecutarla; si cancelas el aviso de UAC la app simplemente no se abre y no se toca nada.
- **No hace falta instalar .NET**: el runtime va incluido en el `.exe` (por eso pesa ~160 MB).
- Conexión a Internet **opcional**: solo para licencias, actualizaciones y el catálogo de winget.

---

## Arquitectura

Sothe Optimizer es una aplicación **WPF sobre .NET 8** publicada como *single-file self-contained*. La lógica está separada en motor, datos e interfaz:

```
┌─────────────────────────── Interfaz (WPF) ───────────────────────────┐
│ Optimizaciones · Apps (winget) · Reparaciones · Características      │
└──────────────┬────────────────────────────────────┬──────────────────┘
               │ aplica / revierte                  │ licencia / actualización
┌──────────────▼───────────────┐      ┌─────────────▼───────────────────────┐
│ Motor (SotheEngine)          │      │ Licencias (Keygen) · AppUpdater     │
│ registro · servicios · cmd   │      │ OTA de datos firmada                │
│ backups · reconciliación     │      └─────────────────────────────────────┘
└──────────────▲───────────────┘
               │ lee (JSON incrustado en el .exe, o actualizado y firmado)
        tweaks · apps · features · profiles
```

### Reversibilidad y reconciliación

- Cada ajuste es una lista de **operaciones** (registro, servicio, comando, paquete Appx) descritas en JSON: añadir un ajuste no toca el código.
- Antes de aplicar, se guarda el **estado previo** en `%ProgramData%\SotheOptimizer\backups`.
- El botón principal **reconcilia**: lo marcado y no aplicado se aplica; **lo desmarcado pero aplicado se revierte**; lo ya aplicado se deja. El sistema queda idéntico a lo que ves marcado.
- Un revertido que **falla conserva el backup** (el ajuste sigue contando como aplicado y se puede reintentar) y queda anotado en `sothe.log`.
- Hay **simulación (dry-run)** para ver qué cambiaría sin tocar nada.

### Timer Resolution (API nativa de Windows)

El ajuste «Timer Resolution fija» pide al kernel una resolución de **0,5 ms** mediante `NtSetTimerResolution` / `NtQueryTimerResolution` de `ntdll.dll` (P/Invoke). El temporizador de plataforma es global: mientras *algún* proceso mantenga la petición, todo el sistema va a esa resolución. Por eso:

- Se registra una **tarea programada al iniciar sesión** que lanza un proceso residente ligero (`--timer-resident`, 0 % de CPU: está bloqueado esperando un evento).
- Un **mutex global** evita instancias duplicadas.
- Al terminar (al revertir el ajuste o cerrar sesión) el residente **suelta su petición de forma explícita** (`NtSetTimerResolution(..., false)`) para que la resolución vuelva a la del sistema y no se gaste batería de más. Al revertir, se avisa por un evento con nombre y solo como último recurso se fuerza el cierre.
- La tarea se ejecuta con privilegios elevados, así que el `.exe` que lanza se **copia a `Archivos de programa`** (carpeta que solo modifica un administrador) y la tarea apunta ahí, nunca a tu carpeta de Descargas.

### Auto-actualización firmada (`AppUpdater`)

1. Al arrancar consulta la última *release* de este repositorio (solo **HTTPS** y solo hosts de GitHub).
2. Descarga `release.json` + `release.sig` y comprueba la firma **ECDSA P‑256** con la **clave pública incrustada** en la app. La clave privada vive solo en el PC del autor.
3. Si la versión es más nueva, descarga el `.exe` **por *streaming*** a disco, calculando el **SHA‑256** al vuelo y con un tope de tamaño; si el hash o el tamaño no coinciden con lo firmado, se descarta y no se guarda nada.
4. Se instala cuando no hay nada en curso (o al siguiente arranque): un script **espera a que el proceso termine, guarda una copia del `.exe` actual, lo reemplaza, lo relanza** y, si algo falla, **restaura la versión anterior**.
5. Sin conexión, con GitHub caído o con datos corruptos: no pasa nada. Un fallo de actualización se anota en el log y la app sigue funcionando.

Aunque GitHub o tu red fueran comprometidos, **sin la clave privada del autor no se puede colar un binario** que la app llegue a ejecutar.

### Licencias

- Validación con **[Keygen](https://keygen.sh)**: licencias *node-locked* (un PC por clave) y pruebas de 3 meses.
- La **huella del equipo** es un SHA‑256 (con sal) del UUID de la BIOS y la serie de la placa; nunca se envían en claro.
- La licencia se guarda **cifrada con DPAPI** y se valida en segundo plano. Hay una **gracia offline de 14 días** y detección de relojes atrasados o con fechas imposibles.
- Los ajustes gratuitos funcionan siempre y **revertir es siempre gratis**.

### Datos actualizables (OTA)

Los JSON con los ajustes pueden actualizarse sin reinstalar, pero como contienen comandos que se ejecutan como administrador, **solo se aceptan si el manifiesto está firmado** con la clave del autor y cada fichero coincide con su SHA‑256. Un JSON sin firmar, editado a mano o más antiguo se ignora y se usa la copia incrustada en el `.exe`.

---

## Seguridad

Sothe Optimizer pide privilegios de administrador y ejecuta comandos, así que se ha auditado como tal (análisis estático del código y 212 pruebas automatizadas). Medidas principales:

| Superficie | Medida |
|---|---|
| **Actualizador** | HTTPS + lista de hosts · firma ECDSA P‑256 · SHA‑256 y tamaño firmados · descarga con tope · reemplazo con copia y marcha atrás |
| **Carpeta de datos** (`%ProgramData%\SotheOptimizer`) | La app solo confía en lo creado por un administrador o el sistema; ACL protegida (los usuarios estándar solo leen); lo plantado por otros se elimina |
| **Ejecución de procesos** | `cmd.exe`, `powershell.exe` y `winget` se lanzan por **ruta absoluta**, para que un ejecutable colocado junto al `.exe` no se ejecute como administrador |
| **Comandos** | Identificadores de winget, Características y nombres de servicio/paquete validados con listas estrictas; `ProcessStartInfo.ArgumentList` en lugar de concatenar argumentos |
| **Datos OTA** | Firma + hash por fichero; solo ficheros de una lista cerrada |
| **Licencias** | Caché cifrada con DPAPI, huella con sal, rechazo de fechas de validación falsas |

**Privacidad.** No hay telemetría. Las únicas conexiones son: GitHub (comprobar actualizaciones), Keygen (licencia: clave, huella cifrada con hash y nombre del equipo) y winget/Microsoft (catálogo de apps) cuando lo usas.

**Límites que conviene saber.** Un programa .NET que corre en tu PC nunca puede protegerse al 100 % de quien lo ejecuta. La app no es una caja negra para quien tenga ganas de analizarla; la protección real de la licencia está en el servidor.

¿Has encontrado una vulnerabilidad? Abre un [aviso de seguridad privado](https://github.com/ImSothe/Sothe-Windows-Optimizer/security/advisories/new) (ver [SECURITY.md](SECURITY.md)).

---

## Desinstalar y deshacer

- Dentro de la app: **Revertir TODO** restaura cada ajuste desde su backup.
- Para quitar la app basta con borrar `SotheOptimizer.exe`. Si activaste *Timer Resolution fija*, **revierte ese ajuste primero** (elimina la tarea programada y su copia en `Archivos de programa`).
- Los datos viven en `%ProgramData%\SotheOptimizer` (backups, perfiles propios, log).

---

## Aviso legal

Sothe Optimizer modifica ajustes profundos de Windows. **Úsalo bajo tu propia responsabilidad**: algunos ajustes desactivan protecciones de seguridad o pueden causar inestabilidad. Crea siempre un punto de restauración antes de aplicar cambios. El software se ofrece «TAL CUAL», sin garantías. Consulta [LICENSE](LICENSE).

---

## English

**Sothe Windows Optimizer** is a Windows 10/11 tuning tool for power users (competitive gaming and security labs). It applies 70+ registry, service and system tweaks, **never hides the dangerous ones** (Defender, firewall, Spectre/Meltdown mitigations…), and makes **every change reversible** by storing the previous value first.

- **Download:** grab `SotheOptimizer.exe` from [Releases](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest). It is a single self-contained file (the .NET 8 runtime is bundled) and needs **administrator rights** (UAC).
- **Integrity:** compare `Get-FileHash .\SotheOptimizer.exe` with the `sha256` in `release.json`. The binary is not Authenticode-signed yet, so SmartScreen may warn (*More info → Run anyway*).
- **Engineering highlights:** WPF/.NET 8 single-file app; `ntdll` P/Invoke for 0.5 ms timer resolution with an explicit release on exit; **ECDSA P-256 signed auto-updater** (HTTPS + host allow-list, streamed SHA-256 verification, size cap, backup-and-rollback swap script); **Keygen node-locked licensing** with DPAPI-protected cache and hashed hardware fingerprint; **signed OTA data**; hardened `%ProgramData%` storage; absolute-path process launching; 212 automated tests.
- **Source code is closed.** This repository only hosts the binary releases.
