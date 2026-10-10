# Política de seguridad

## Versiones con soporte

Solo la **última release** recibe correcciones de seguridad. La app se actualiza sola (actualizaciones firmadas), así que lo normal es que ya la tengas.

## Cómo comprobar que lo que descargas es auténtico

1. Descarga `SotheOptimizer.exe` solo desde la [página de Releases de este repositorio](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases).
2. Compara el SHA‑256: `Get-FileHash .\SotheOptimizer.exe -Algorithm SHA256` debe coincidir con `sha256` en `release.json` de esa misma release.
3. Cada release incluye `release.sig`, firma **ECDSA P‑256** de `release.json`. La app la verifica con una clave pública incrustada antes de instalar cualquier actualización.

## Cómo informar de una vulnerabilidad

Por favor **no abras un *issue* público**. Usa un [aviso de seguridad privado de GitHub](https://github.com/ImSothe/Sothe-Windows-Optimizer/security/advisories/new) con:

- Qué has encontrado y el impacto.
- Pasos para reproducirlo (versión de la app y de Windows).
- Si es posible, una propuesta de solución.

Respondo lo antes que puedo y te cito en las notas de la corrección si quieres.

## Alcance

Dentro: el ejecutable publicado, su auto-actualizador, el sistema de licencias y el manejo de la carpeta `%ProgramData%\SotheOptimizer`.
Fuera: que la herramienta modifique ajustes de seguridad de Windows cuando **tú lo pides** (es su función y está avisado en el EULA), y ataques que ya requieran ser administrador del equipo.
