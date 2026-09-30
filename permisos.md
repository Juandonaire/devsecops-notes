# Informe de Práctica: Gestión de Permisos y Control de Acceso en Linux

## 1. Mapeo de Permisos Octales y Simbólicos
- `644` (`rw-r--r--`): Propietario lectura/escritura; Grupo y Otros solo lectura.
- `750` (`rwxr-x---`): Propietario acceso total; Grupo lectura/ejecución; Otros sin acceso.
- `700` (`rwx------`): Acceso exclusivo para el propietario.

## 2. Comportamiento del Bit de Ejecución (`x`)
- **Archivos:** Requisito obligatorio para la ejecución de scripts/binarios (`./script.sh`). Sin él, Bash devuelve `Permission denied`.
- **Directorios:** Actúa como permiso de paso/travesía. Sin el bit `x`, no es posible acceder con `cd` ni consultar el contenido.

## 3. Carpetas Compartidas y Bit SETGID (`g+s`)
- **Permisos Base (`770`):** Aísla la carpeta restringiendo el acceso exclusivamente al propietario y al grupo asignado.
- **SETGID (`2770` / `chmod g+s`):** Garantiza la colaboración forzando que todo archivo o subdirectorio creado herede el grupo propietario de la carpeta padre, evitando bloqueos de permisos entre miembros del equipo.

## 4. Auditoría con `find`
- Identificación de propiedad: `find /ruta -user <usuario>`
- Detección de bits especiales (SETGID/SUID): `find /ruta -perm -2000`
- Auditoría de vulnerabilidades (World-Writable): `find /ruta -perm -0002`
