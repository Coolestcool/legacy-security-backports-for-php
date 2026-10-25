# Guía de aplicación para PHP 5.6.40 y 7.2.34

Este repositorio distribuye los backports de Sury para CVE-2026-109618 y
CVE-2026-109619, con créditos originales conservados. Este repositorio es una redistribución
comunitaria mantenida por **Coolestcool**, sin atribuirse la autoría de los
parches ni presentarse como proyecto oficial de Sury.

Son mitigaciones para versiones sin soporte; no equivalen a una versión moderna
mantenida ni a cubrir todas las vulnerabilidades. Conservan el número de PHP,
terminan inmediatamente el proceso ante ciertos errores de memoria y desactivan
el seguimiento del progreso de subidas. Las subidas normales deben seguir
funcionando y se prueban por separado.

## Descargar y preparar

```sh
git clone --branch v0.1.1 https://github.com/Coolestcool/legacy-security-backports-for-php.git
cd legacy-security-backports-for-php
sha256sum -c SHA256SUMS

# Sólo comprobar: usa copias temporales, no modifica las fuentes entregadas.
python3 tools/prepare-source.py /ruta/php-5.6.40
# O, para PHP 7.2.34:
python3 tools/prepare-source.py /ruta/php-7.2.34

# Aplicar explícitamente a UNA copia de trabajo de la versión correspondiente:
python3 tools/prepare-source.py /ruta/php-5.6.40 --apply
# O:
python3 tools/prepare-source.py /ruta/php-7.2.34 --apply
```

En macOS usar `shasum -a 256 -c SHA256SUMS`. Se necesita Python 3.8+, `patch`
y un directorio temporal escribible. El prerrequisito ZSTR_MAX_LEN de 7.2 se
aplica automáticamente cuando faltan ambas macros; si ambas existen se omite.
Se rechazan versiones inesperadas, macros incompletas, hashes distintos,
contextos incompatibles o parches principales ya aplicados. No usar `--force`
ni ignorar rechazos.

Cada parche principal aborda **las dos CVE** para su versión de PHP. No aplicar
el parche de 5.6 a 7.2 ni ambos al mismo árbol. Para paquetes como `5.6.40-38`,
obtener las fuentes de esa revisión y conservar todos los parches originales
del distribuidor. Comprobar primero en una copia aislada.

## Compilar y desplegar

**Modificar las fuentes no modifica el servidor en ejecución.** Seguir la
[guía técnica completa](INSTALL.md) para compilar paquetes o imágenes con las
mismas extensiones, bibliotecas, flags, arquitectura y rutas de configuración.
La compilación mínima de CI es exclusivamente de laboratorio.

1. Identificar PHP real en CLI, CGI, Apache, FPM y contenedores. `php -v` sólo
   acredita el CLI seleccionado; no prueba qué ejecuta la web.
2. Respaldar binarios, módulo Apache, extensiones, configuración, permisos e
   imagen anterior. Probar su restauración en una copia y mantener acceso a consola.
3. Compilar en un entorno compatible y probar cada SAPI usado, extensiones,
   sesiones, subidas y flujos de aplicación con datos de pruebas.
4. Desplegar mediante el procedimiento de paquetes/imágenes de cada sistema.
   Reiniciar o retirar todos los workers antiguos y verificar los ejecutables y
   módulos realmente cargados. Comprobar HTTP y logs nuevamente.
5. Si falla un criterio, restaurar paquetes/binarios/configuración e imagen
   anteriores y verificar la recuperación. Revertir sólo el archivo .patch no
   revierte binarios instalados. El rollback también recupera la exposición anterior.

No ejecutar pruebas de agotamiento de memoria, pedidos, pagos o envíos de correo
sobre producción. Las pruebas incluidas sólo usan procesos CLI/CGI locales,
no prueban Apache/FPM ni todos los flujos. No publicar configuraciones, datos de
usuarios o credenciales en issues.

La configuración puede mostrar `session.upload_progress.enabled=1` aunque el
backport desactive esa función directamente en código. Verificar el
comportamiento en laboratorio; no basta leer php.ini.

Consulta [validación y límites](VALIDATION.md), [procedencia](../PROVENANCE.md),
[seguridad](../SECURITY.md) y [contribuciones](../CONTRIBUTING.md).
