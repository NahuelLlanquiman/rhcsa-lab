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
