# Despliegue automatizado de Moodle + CodeRunner

Proyecto de Trabajo de Fin de Grado (ESEI, Universidade de Vigo, 2025):
*Desarrollo e Integración de CodeRunner en Moodle para la Evaluación Automatizada de Ejercicios de Programación*.

Este repositorio contiene la parte de **infraestructura como código** del TFG: un playbook de
Ansible que levanta, desde cero y sin pasos manuales, una plataforma Moodle con el plugin
[CodeRunner](https://coderunner.org.nz) y un sandbox [Jobe](https://github.com/trampgeek/jobe),
para corregir automáticamente ejercicios de programación (en el TFG, de Java para la asignatura
Algoritmos y Estructuras de Datos I).

> **Estado:** proyecto académico de demostración. Las contraseñas incluidas son de ejemplo y
> **no deben usarse en producción**.

## Demostración

Vídeos de algunas pruebas del sistema: 
Vídeo de demostración de la creación de un cuestionario: https://youtu.be/M4l2daK2vIg
Vídeo de demostración de la realización y autocorrección de un cuestionario: https://youtu.be/yetVJz6Qj18

## Arquitectura

Tres contenedores Docker en una red privada, orquestados con `docker compose` y desplegados por Ansible:

```
 navegador ──► :8080 ┌──────────────────────────┐
                     │ moodle  (Moodle 4.5 +    │
                     │          CodeRunner)     │
                     └───────┬──────────┬───────┘
                             │          │
                      ┌──────▼───┐  ┌───▼────────────────┐
                      │ db       │  │ jobe               │
                      │ MySQL 8  │  │ sandbox de ejecución│
                      └──────────┘  └────────────────────┘
                        (red interna `moodle_net`)
```

- Sólo Moodle es accesible desde fuera (puerto `8080`).
- La base de datos únicamente es accesible desde Moodle.
- Jobe (donde se ejecuta el código del alumnado) únicamente acepta peticiones de Moodle.
- Los datos (base de datos y `moodledata`) se guardan en volúmenes persistentes.

## Estructura del repositorio

| Ruta | Contenido |
|---|---|
| `playbook.yml` | Playbook de Ansible: instala Docker, levanta el stack, instala Moodle, CodeRunner y crea curso, usuarios y cuestionarios |
| `setup.yml` | Variables (usuarios, contraseñas de ejemplo, nombres del curso y los cuestionarios...) |
| `requirements.yml` | Colecciones de Ansible necesarias (`community.docker`) |
| `hosts`, `ansible.cfg` | Inventario y configuración de Ansible |
| `moodle_project/docker-compose.yml` | Definición de los servicios Moodle, MySQL y Jobe |
| `moodle_project/moodle/Dockerfile` | Imagen de Moodle (PHP 8.1 + Moodle 4.5) |

## Requisitos

- Una **máquina anfitriona** (en el desarrollo, Ubuntu) con Python 3 y acceso a Internet.
- Una **máquina virtual Debian 12** accesible por SSH desde la anfitriona con el puerto **2222**
  reenviado (`127.0.0.1:2222`). El usuario por defecto es `user` (ver `hosts`).
  No es imprescindible usar estos sistemas operativos, pero sí que la VM sea accesible así
  (o modificar `hosts` a tu entorno).

## Uso

Desde la raíz del repositorio, en la máquina anfitriona:

1. Instalar Ansible (con permisos de administrador para los paquetes del sistema):

   ```bash
   sudo apt update && sudo apt install -y python3-pip python3-venv sshpass
   python3 -m venv ~/venv-ansible
   source ~/venv-ansible/bin/activate
   pip install ansible requests
   ansible-galaxy install -r requirements.yml
   ```

2. Ejecutar el playbook:

   ```bash
   ansible-playbook -i hosts playbook.yml
   ```

   Pedirá dos contraseñas (SSH y `sudo`), que por defecto son la misma: `user`.
   La ejecución **tarda bastante** (descarga imágenes, clona Moodle e instala plugins);
   no significa que se haya colgado.

3. Abrir <http://localhost:8080/moodle>.

### Usuarios creados (sólo para demo local)

| Rol | Usuario | Contraseña |
|---|---|---|
| Administrador | `admin` | `Admin123!` |
| Profesor | `profesor` | `profesor` |
| Alumno | `alumno` | `alumno` |

Se pueden cambiar en `setup.yml` antes de ejecutar el playbook. El profesor y el alumno
quedan matriculados automáticamente en el curso de muestra.

## Curso de muestra y preguntas

El playbook crea un curso (`TFG_lcm`) con dos cuestionarios vacíos: *Estructuras Enlazadas* y
*TADs (Pilas, Colas, Listas y Conjuntos)*.

**El banco de preguntas del TFG no se incluye en este repositorio**, porque contiene los
ejercicios con sus soluciones. Para ver el sistema funcionando se pueden crear preguntas de tipo
CodeRunner desde Moodle (*Banco de preguntas → Crear una nueva pregunta → CodeRunner*) o importar
un fichero propio en formato Moodle XML:

1. Activar el *Modo edición* en el curso.
2. Ir al *Banco de preguntas* → *Importar* → formato **Moodle XML** → subir el fichero.
3. En el cuestionario, *Agregar → del banco de preguntas*, filtrar por categoría, seleccionar las
   preguntas y añadirlas.

## Notas de seguridad

- Las contraseñas de `setup.yml`, `docker-compose.yml` y la clave de API de Jobe incluida en el
  playbook (la clave por defecto de la imagen `jobeinabox`) son valores de ejemplo.
  Cámbialos antes de cualquier uso fuera de un entorno de pruebas.
- Jobe no publica ningún puerto al exterior: sólo es accesible desde Moodle por la red interna.

## Licencia

Código bajo licencia [MIT](LICENSE). Moodle y CodeRunner son software de terceros (GPL) y
se descargan durante el despliegue; no se redistribuyen en este repositorio.
