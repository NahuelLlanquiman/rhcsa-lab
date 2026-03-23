# Ejercicios — Gestión de archivos

Comandos ejecutados en Rocky Linux 10.1 sobre VirtualBox.
Cada ejercicio incluye el comando real, su output y el aprendizaje obtenido.

---

## Clase 1 — Navegación del sistema de archivos

### Exploración inicial
```bash
pwd
# Output: /home/admin
# pwd muestra el directorio actual — esencial para ubicarse en el sistema

ls -la
# Muestra contenido con permisos, dueño, tamaño y fecha
# La d al inicio indica directorio, - indica archivo regular
# Los archivos que empiezan con . son ocultos

cd /
ls
# Directorios vistos: afs bin boot dev etc home lib lib64
# media mnt opt proc root sbin srv sys tmp usr var
```

### Archivos de configuración clave
```bash
cat /etc/hostname
# rocky-lab — nombre configurado con hostnamectl

cat /etc/hosts
# 127.0.0.1 localhost — loopback, siempre apunta a la propia maquina
# Las lineas con # son comentarios, el sistema las ignora

cat /etc/host.conf
# multi on — permite que un hostname tenga multiples IPs
```

### Configurar hostname del servidor
```bash
sudo hostnamectl set-hostname rocky-lab
# Requiere sudo porque modifica configuracion del sistema
# Cambio aplicado de inmediato sin reiniciar
# El prompt cambio de admin@localhost a admin@rocky-lab
```

### Aprendizaje clave
El sistema de archivos de Linux parte desde / (raiz).
Todo — discos, dispositivos, configuraciones — vive dentro de ese arbol.
/etc contiene toda la configuracion del sistema.

---

## Clase 2 — Manipulacion de archivos

### Crear y escribir archivos
```bash
touch notas.txt servidor.conf log_prueba.log
# Crea archivos vacios — si ya existen actualiza su fecha

echo "Mi primer servidor Rocky Linux" > notas.txt
# > sobreescribe el archivo completamente — peligroso en produccion

echo "Fecha de inicio: marzo 2026" >> notas.txt
# >> agrega al final sin borrar el contenido existente

cat notas.txt
# Output:
# Mi primer servidor Rocky Linux
# Fecha de inicio: marzo 2026
```

### Copiar, mover y renombrar
```bash
cp notas.txt backups/notas_backup.txt
# Copia el archivo con nuevo nombre dentro de backups/

mv log_prueba.log log_sistema.log
# mv con nombre nuevo = renombra (mismo directorio)

mv servidor.conf scripts/
# mv con directorio como destino = mueve el archivo
```

### Eliminar archivos y directorios
```bash
rm basura1.txt
# Elimina archivo — sin papelera de reciclaje, no hay recuperacion

rm dir_vacio
# Error: rm no borra directorios sin el flag -r
# Mensaje: Es un directorio

rmdir dir_vacio
# rmdir solo funciona con directorios vacios

rm -r dir_prueba
# -r borra recursivamente el directorio y todo su contenido
```

### Aprendizaje clave

La diferencia entre > y >> puede destruir un archivo de configuracion
en produccion. Siempre hacer backup antes de modificar archivos criticos.
rm -rf es irreversible — usarlo con extrema precaucion.
---

## Clase 3 — Busqueda con find

### Busquedas basicas
```bash
find / -name "hostname" 2>/dev/null
# Encontro 6 ubicaciones distintas del mismo dato:
# /etc/hostname — archivo de configuracion
# /usr/bin/hostname — el comando ejecutable
# /proc/sys/kernel/hostname — el kernel en memoria
# 2>/dev/null descarta errores de "permiso denegado"

find /etc -name "*.conf" 2>/dev/null
# Lista todos los archivos de configuracion del sistema
# El * es comodin — representa cualquier texto

find ~/laboratorio -type d
# -type d filtra solo directorios

find ~/laboratorio -type f
# -type f filtra solo archivos regulares
```

### Busquedas avanzadas
```bash
find /var -size +1M 2>/dev/null
# Encontro archivos grandes:
# /var/lib/dnf/history.sqlite-wal — historial de paquetes instalados
# /var/cache/dnf/*.xml.gz — cache de repositorios
# /var/log/anaconda/journal.log — log de instalacion del sistema

find ~/laboratorio -mmin -60
# Archivos modificados hace menos de 60 minutos
# -mmin -1440 = ultimas 24 horas (1440 minutos)

find ~/laboratorio -empty
# Encontro archivos vacios creados con touch sin contenido
```

### find con acciones
```bash
find ~/laboratorio -type f -ls
# Muestra detalles completos incluyendo numero de inode
# El inode es el identificador unico real del archivo en Linux

find ~/laboratorio -maxdepth 1 -name "*.txt" -exec cp {} ~/laboratorio/backups/ \;
# -maxdepth 1 limita la busqueda al directorio actual sin entrar subcarpetas
# -exec ejecuta un comando por cada archivo encontrado
# {} representa el archivo encontrado en cada iteracion
# \; indica el fin del comando a ejecutar

find ~/laboratorio -type f -printf "%f\n"
# -printf formato personalizado de salida
# %f imprime solo el nombre sin la ruta completa
# %p imprimiria la ruta completa
# \n agrega salto de linea despues de cada resultado
```

### Aprendizaje clave
find busca en tiempo real recorriendo el arbol de directorios.
Los inodes son los identificadores reales de archivos en Linux —
el nombre es solo una etiqueta que apunta al inode.
2>/dev/null es una redireccion de errores — el 2 representa stderr.

## Clase 4 -Vim, el editor del sysadmin

## Por que Vim y no otro editor

Vim esta disponible en absolutamente en todos los sistemas Linux y Unix.
En servidores remotos sin interfaz grafica es el unico editor disponible.
El examen RHCSA solo permite Vim - dominarlo es obligatorio.

### Los tres modos fundamentales

Modo Normal: donde arranca Vim siempre. Las teclas ejecutan comandos.
Modo Insercion: donde se escribe texto. Se entra con i y se sale con ESC.
Modo Comando: donde se guarda, sale y busca. Se entra con : desde Normal.
Regla absoluta: ESC siempre lleva de vuelta al Modo Normal.

### Navegacion en Modo Normal
``` bash

h j k l  — izquierda, abajo, arriba, derecha
0        — inicio de linea
$        — final de linea
gg       — primera linea del archivo
G        — ultima linea del archivo
w        — siguiente palabra
b        — palabra anterior
:N       — ir directamente a la linea N
Ctrl+g   — ver linea actual y total de lineas
numero+comando — repetir comando N veces (5j baja 5 lineas, 4dd elimina 4)
```
### Edicion en Modo Normal
```bash
dd  — elimina linea completa (queda en portapapeles de Vim)
yy  — copia linea completa (yank)
p   — pega debajo de la linea actual
P   — pega encima de la linea actual
u   — deshace el ultimo cambio
Ctrl+r — rehace lo que se deshizo
x   — elimina el caracter bajo el cursor
r   — reemplaza un solo caracter
o   — abre linea nueva abajo y entra a insercion
O   — abre linea nueva arriba y entra a insercion
A   — va al final de la linea y entra a insercion
I   — va al inicio de la linea y entra a insercion

Importante: mayuscula != minuscula en Vim
p pega debajo, P pega encima
o abre abajo, O abre arriba
s sustituye caracter, S elimina linea entera
```
### Busqueda
``` bash
/palabra  — busca hacia adelante
?palabra  — busca hacia atras
n         — siguiente coincidencia
N         — coincidencia anterior
*         — busca la palabra bajo el cursor
:noh      — apaga el resaltado de busqueda
```

### Reemplazo — estructura del comando
```bash
:[rango]s/[buscar]/[reemplazar]/[opciones]

Rangos:
  sin rango = solo linea actual donde esta el cursor
  %         = todo el archivo

Opciones:
  sin opcion = primera ocurrencia de la linea actual
  g          = todas las ocurrencias del archivo sin preguntar
  gc         = todas las ocurrencias preguntando una por una (y/n/a/q/l)
  i          = ignora mayusculas al buscar

Ejemplos reales:
  :s/^#PermitRootLogin/PermitRootLogin/   <- solo linea actual
  :%s/Rocky/RHEL/g                        <- todo el archivo sin preguntar
  :%s/Rocky/RHEL/gc                       <- todo el archivo preguntando
  :%s/^#Puerto/Puerto/                    <- el ^ ancla al inicio de linea
```
### Guardar y salir
```bash
:w   — guardar sin salir
:q   — salir (solo si no hay cambios)
:wq  — guardar y salir
:q!  — salir sin guardar (el ! fuerza la salida)
```
### Configuracion personal — .vimrc

~/.vimrc es el archivo de configuracion personal de Vim.
Se carga automaticamente cada vez que abres Vim.
Con sudo vim hay que copiar el .vimrc al home de root:
sudo cp ~/.vimrc /root/.vimrc

Configuraciones utiles aplicadas:
set number         — numeros de linea absolutos
set relativenumber — numeros relativos al cursor para saltar lineas rapido
set hlsearch       — resalta coincidencias de busqueda
set incsearch      — busqueda incremental mientras escribes
syntax on          — colores de sintaxis segun tipo de archivo

### Aprendizaje clave

En Vim mayuscula y minuscula son comandos completamente distintos.
Un error comun es entrar a modo insercion sin darse cuenta con o, O, A, I.
La solucion siempre es ESC — vuelve al Modo Normal desde cualquier estado.
El .vimrc convierte Vim en un editor personalizado — en el RHCSA puedo
configurarlo antes de empezar para trabajar mas comodo.
El rango determina donde opera el reemplazo — sin % solo la linea actual,
con % todo el archivo.
