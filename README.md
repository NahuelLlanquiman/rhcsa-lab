# rhcsa-lab

Laboratorio personal de preparación para el examen RHCSA (EX200) de Red Hat.
Todo el trabajo está hecho en Rocky Linux 10 sobre VirtualBox.

## Descripción

Este repositorio es mi cuaderno técnico público. Cada carpeta corresponde
a un dominio del examen RHCSA. Cada archivo documenta comandos reales
ejecutados en mi VM, errores encontrados y aprendizajes del proceso.

## Tecnologías utilizadas

- Rocky Linux 10.1 (Red Quartz)
- Kernel 6.12.x
- VirtualBox 7.x
- Bash 5.x / Vim 9.x / Git 2.47.x

## Requisitos para reproducir

- VirtualBox 7.x
- ISO Rocky Linux 10 (rockylinux.org)
- VM con mínimo 2GB RAM y 20GB disco

## Estructura del repositorio

| Carpeta | Contenido |
|---|---|
| 01-gestion-archivos | find, cp, mv, rm, vim |
| 02-usuarios-permisos | chmod, chown, ACLs |
| 03-servicios-systemd | systemctl, journalctl |
| 04-redes | nmcli, ss, firewalld |
| 05-almacenamiento-lvm | fdisk, pvcreate, lvcreate |
| 06-selinux | contextos, políticas, troubleshooting |
| 07-contenedores-podman | imágenes, volúmenes, pods |
| scripts | scripts bash reutilizables |

## Aprendizajes clave

- La diferencia entre `>` y `>>` puede destruir un archivo de configuración en producción
- `systemctl enable` y `systemctl start` hacen cosas distintas
- SELinux bloquea silenciosamente — siempre revisar antes de desactivarlo
