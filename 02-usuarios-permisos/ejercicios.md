## Clase 5 — Permisos de archivos

### Estructura de permisos en Linux

``` bash

Cada archivo tiene un bloque de 10 caracteres que define sus permisos:



- rw-  r–  r–
│└───┘└──┘└──┘
│  │    │   └── others: todos los demas usuarios
│  │    └─────── group: el grupo del archivo
│  └──────────── owner: el dueño del archivo
└─────────────── tipo: - archivo, d directorio


Cada grupo tiene tres posibles permisos:
r = 4 = leer
w = 2 = escribir
x = 1 = ejecutar
- = 0 = sin permiso

```
### Permisos numericos mas usados
``` bash

644 = rw-r--r-- archivos normales (dueño lee/escribe, resto solo lee)
755 = rwxr-xr-x scripts y directorios (dueño todo, resto lee/ejecuta)
600 = rw------- archivos privados (solo dueño lee y escribe)
700 = rwx------ directorios privados (solo dueño entra)

```
### chmod — cambiar permisos
```bash

# Forma simbolica — cambios puntuales
chmod o-r archivo.txt      # quitar lectura a others
chmod u+x script.sh        # agregar ejecucion al dueño
chmod a+r archivo.txt      # agregar lectura a todos

# Forma numerica — establecer permisos exactos
chmod 644 archivo.txt      # rw-r--r--
chmod 755 script.sh        # rwxr-xr-x
chmod 600 privado.txt      # rw-------

```

### chown — cambiar dueño y grupo
```bash

# Cambiar solo el grupo (requiere sudo si no eres miembro del grupo)
sudo chown :root archivo.txt

# Cambiar dueño y grupo a la vez
sudo chown root:root archivo.txt

# Volver al estado original
sudo chown admin:admin archivo.txt

```
Aprendizaje Clave
Aprendi que sin sudo solo puedo cambiar el grupo a uno donde
yo sea miembro. Para cambiar a grupos ajenos necesito sudo.
id y groups — ver identidad del usuario



```bash

# id
uid=1000(admin) gid=1000(admin) grupos=1000(admin),10(wheel)
contexto=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# groups
admin wheel

```

El grupo wheel es especial — pertenecer a el otorga acceso a sudo.

El contexto SELinux aparece al final de id — lo vere en Fase 3.
uid=0 siempre es root. Los usuarios normales empiezan en uid=1000.



### Permisos especiales — SUID

Descubri la s en /usr/bin/sudo:
```bash

—s–x–x. 1 root root /usr/bin/sudo

```

SUID (Set User ID) permite ejecutar un archivo con la identidad
del dueño, no del usuario que lo ejecuta. Como sudo es dueño root
y tiene SUID, cuando ejecuto sudo el proceso corre como root
aunque yo sea admin. Sin SUID sudo simplemente no funcionaria.

umask — permisos por defecto
```bash

umask        # ver umask actual: 0022
umask 077    # cambiar temporalmente (solo dura la sesion actual)

```

La umask se resta del maximo posible:
```bash

666 - 022 = 644 para archivos nuevos
777 - 022 = 755 para directorios nuevos

```

Para hacerla permanente hay que agregarla a ~/.bashrc o ~/.bash_profile.
La umask 077 crea archivos con 600 y directorios con 700 —
maxima privacidad, solo el dueño tiene acceso.

Aprendizaje clave

Los permisos en Linux son la primera linea de defensa de seguridad.
Entender que cada usuario cae en una de tres categorias
(owner, group, others) me permite razonar cualquier escenario
sin memorizar casos especificos.

El archivo ~/.ssh/id_ed25519 tiene permisos 600 automaticamente —
SSH se niega a usar claves privadas con permisos mas abiertos
por considerarlas inseguras.

---

## Clase 6 — Usuarios y grupos

### Archivos clave del sistema de usuarios
```bash
/etc/passwd   # informacion de todos los usuarios — 7 campos separados por :
/etc/shadow   # contrasenas encriptadas — solo root puede leerlo
/etc/group    # informacion de todos los grupos
/etc/skel/    # plantilla base copiada al home de cada usuario nuevo
```

Estructura de /etc/passwd:
usuario:x:UID:GID:nombre_completo:home:shell
- x significa que la contrasena esta en /etc/shadow
- UID 0 = root, 1-999 = sistema, 1000+ = usuarios humanos
- /sbin/nologin como shell = cuenta de servicio, no puede hacer login

### useradd — crear usuarios
```bash
sudo useradd -m -s /bin/bash juan
# -m crea /home/juan y copia archivos base desde /etc/skel/
# -s /bin/bash define bash como shell — sin esto puede quedar con sh

sudo useradd -m -s /bin/bash -u 1050 carlos
# -u 1050 asigna UID especifico — util en el examen RHCSA
```

### passwd — asignar contrasenas
```bash
sudo passwd juan
# Rocky Linux rechaza contrasenas debiles:
# - menos de 8 caracteres
# - que contengan el nombre del usuario
# - demasiado simples
# Las contrasenas se guardan encriptadas en /etc/shadow con yescrypt ($y$)
```

### groupadd — crear grupos
```bash
sudo groupadd desarrolladores         # GID asignado automaticamente
sudo groupadd -g 1100 sistemas        # -g asigna GID especifico
grep sistemas /etc/group              # verificar: sistemas:x:1100:
```

### usermod — modificar usuarios
```bash
sudo usermod -aG desarrolladores juan
# -a = append — agrega sin quitar grupos existentes
# -G = grupos secundarios
# SIN -a reemplaza todos los grupos — puede quitar acceso a sudo

sudo usermod -L juan    # bloquea cuenta — agrega ! al hash en /etc/shadow
sudo usermod -U juan    # desbloquea — quita el !
sudo usermod -s /sbin/nologin juan   # cambia shell a nologin
sudo usermod -s /bin/bash juan       # vuelve a bash
```

### userdel — eliminar usuarios
```bash
sudo userdel juan       # elimina usuario pero deja /home/juan intacto
sudo userdel -r maria   # -r elimina usuario Y su home directory
# Sin -r el home queda huerfano — el dueño aparece como numero (UID)
# en produccion se preservan los homes por politica de empresa
```

### Verificacion — siempre al final de cada tarea
```bash
id carlos                    # uid, gid y grupos del usuario
groups carlos                # solo los grupos
grep carlos /etc/passwd      # entrada completa en passwd
grep carlos /etc/shadow      # hash de contrasena (requiere sudo)
grep sistemas /etc/group     # miembros del grupo
ls -la /home/                # verificar que el home existe
```

### Escenario RHCSA practicado

Tarea: crear usuario carlos con UID 1050, grupo sistemas con GID 1100,
agregar carlos al grupo, asignar contrasena.

Resultado verificado:
uid=1050(carlos) gid=1050(carlos) grupos=1050(carlos),1100(sistemas)
sistemas:x:1100:carlos

### Aprendizaje clave

El flag -aG en usermod es critico — sin la -a se reemplazan todos
los grupos del usuario y puede perder acceso a sudo instantaneamente.
/etc/skel/ es la plantilla de todo usuario nuevo — si necesito que
todos los usuarios tengan cierta configuracion por defecto la agrego
ahi antes de crearlos.
Las cuentas bloqueadas con -L no pierden su contrasena — el !
es solo una bandera reversible. Util para suspender acceso
temporalmente sin eliminar la cuenta.
