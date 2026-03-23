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

###chown — cambiar dueño y grupo
```bash

# Cambiar solo el grupo (requiere sudo si no eres miembro del grupo)
sudo chown :root archivo.txt

# Cambiar dueño y grupo a la vez
sudo chown root:root archivo.txt

# Volver al estado original
sudo chown admin:admin archivo.txt

```
#Aprendizaje Clave
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



###Permisos especiales — SUID

Descubri la s en /usr/bin/sudo:
```bash

—s–x–x. 1 root root /usr/bin/sudo

```

SUID (Set User ID) permite ejecutar un archivo con la identidad
del dueño, no del usuario que lo ejecuta. Como sudo es dueño root
y tiene SUID, cuando ejecuto sudo el proceso corre como root
aunque yo sea admin. Sin SUID sudo simplemente no funcionaria.

###umask — permisos por defecto
```bash

umask        # ver umask actual: 0022
umask 077    # cambiar temporalmente (solo dura la sesion actual)

```

###La umask se resta del maximo posible:
```bash

666 - 022 = 644 para archivos nuevos
777 - 022 = 755 para directorios nuevos

```

Para hacerla permanente hay que agregarla a ~/.bashrc o ~/.bash_profile.
La umask 077 crea archivos con 600 y directorios con 700 —
maxima privacidad, solo el dueño tiene acceso.

###Aprendizaje clave

Los permisos en Linux son la primera linea de defensa de seguridad.
Entender que cada usuario cae en una de tres categorias
(owner, group, others) me permite razonar cualquier escenario
sin memorizar casos especificos.

El archivo ~/.ssh/id_ed25519 tiene permisos 600 automaticamente —
SSH se niega a usar claves privadas con permisos mas abiertos
por considerarlas inseguras.

---
