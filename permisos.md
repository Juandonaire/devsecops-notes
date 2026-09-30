# Informe de Práctica: Gestión de Permisos, Usuarios y Auditoría en Linux

## 1. Mapeo de Permisos Octales y Simbólicos
- `644` (`rw-r--r--`): Propietario lectura/escritura; Grupo y Otros solo lectura.
- `750` (`rwxr-x---`): Propietario acceso total; Grupo lectura/ejecución; Otros sin acceso.
- `700` (`rwx------`): Acceso exclusivo para el propietario.

## 2. Gestión de Usuarios, Grupos y Propiedad
- **Creación de usuarios y grupos:**
  - `sudo adduser alumno1` (Crea el usuario `alumno1` y su grupo primario).
  - `sudo groupadd proyecto` (Crea el grupo secundario/compartido `proyecto`).
  - `sudo usermod -aG proyecto alumno1` (Añade `alumno1` al grupo `proyecto`).
- **Cambio de propiedad (`chown`):**
  - `sudo chown root:proyecto /srv/proyecto` (Asigna como propietario al usuario `root` y como grupo propietario a `proyecto`).

## 3. Comportamiento y Matices del Bit de Ejecución (`x`)
- **Archivos:** Requisito obligatorio para ejecutar binarios o scripts (`./script.sh`). Sin el bit `x`, la shell devuelve `Permission denied`.
- **Directorios:**
  - **Solo Lectura (`r--` sin `x`):** Permite listar los nombres de los archivos (`ls`), pero genera errores al consultar detalles (`ls -l`) y bloquea la travesía (`cd`) o el acceso a los archivos interiores.
  - **Solo Ejecución (`--x` sin `r`):** Permite atravesar el directorio (`cd`) y acceder o editar un archivo si se conoce su ruta o nombre exacto, pero no permite listar el contenido del directorio (`ls`).

## 4. Carpetas Compartidas, Bit SETGID (`g+s`) y el Rol de la Umask
- **Permisos Base (`770`):** Aísla la carpeta restringiendo el acceso exclusivamente al propietario y al grupo asignado (`drwxrwx---`).
- **SETGID (`2770` / `chmod g+s`):** Fuerza que cualquier archivo o subdirectorio creado herede el grupo propietario del directorio padre.
- **Matiz crucial con la `umask`:** El bit SETGID no otorga permisos de escritura por sí solo. Si la `umask` del usuario es la predeterminada (`022`), los archivos nuevos se crearán con permisos `644` (`rw-r--r--`). Esto significa que otros miembros del grupo podrán leer el archivo, pero **no modificarlo**. Para conseguir una colaboración real en el grupo, es necesario definir una `umask 002` (permisos `664`) o ajustar explícitamente los permisos tras la creación.

## 5. Consultas de Propiedad y Auditoría con `find`

### Consultas de Propiedad
- `find /ruta -user alumno1`: Localiza archivos o directorios cuya propiedad pertenece al usuario `alumno1`.

### Auditoría de Bits Especiales (SUID y SGID)
- **Solo SUID:** `find /ruta -perm -4000` (Busca elementos con el bit SUID activo).
- **Solo SGID:** `find /ruta -perm -2000` (Busca elementos con el bit SETGID activo).
- **Cualquiera de los dos (SUID o SGID):** `find /ruta -perm /6000` (Busca elementos que tengan activo **al menos uno** de los dos bits).

### Auditoría de Vulnerabilidades (World-Writable)
- **Comando:** `find /ruta -type f -perm -0002 2>/dev/null`
- **Riesgo de seguridad:** Un archivo *world-writable* (con permiso de escritura para Otros) puede ser modificado por cualquier usuario o proceso no privilegiado del sistema. Si un proceso con privilegios elevados (`root` o un servicio de sistema) lee o ejecuta ese archivo, un atacante podría alterar su contenido para escalar privilegios o inyectar código malicioso.
