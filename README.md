
### 🎯 **Objetivo General**

Implementar una solución de automatización con **Ansible** que permita administrar y configurar de forma centralizada dos entornos distintos —**Laboratorio Académico** y **Game Center**—, optimizando tareas críticas de administración del sistema operativo, usuarios, servicios, almacenamiento y políticas de software.

---

### 🧠 **Descripción General**

Este proyecto simula la gestión de dos salones informáticos dentro de una institución:

| Salón             | Equipos                                   | Propósito                                                                |
| ----------------- | ----------------------------------------- | ------------------------------------------------------------------------ |
| **Lab Académico** | Servidor + PCs con Windows, Linux y macOS | Uso educativo, software restringido y políticas de seguridad reforzadas. |
| **Game Center**   | Servidor + PCs con Windows, Linux y macOS | Uso recreativo, con acceso a juegos y software multimedia.               |

El administrador (mediante Ansible) puede desplegar configuraciones automáticas, aplicar políticas según el tipo de salón y mantener la seguridad y estabilidad de los equipos.

---

### ⚙️ **Estructura del Proyecto**

```
PROYECTO-ANSIBLE/
│
├── ansible.cfg                     # Configuración principal de Ansible
├── inventory/
│   └── hosts.yml                   # Inventario de hosts y agrupación lógica
├── group_vars/
│   ├── all.yml
│   ├── game_center.yml
│   ├── lab_academico.yml
│   ├── linux.yml
│   └── windows.yml
│
├── roles/
│   ├── common/
│   │   └── tasks/main.yml
│   ├── security_hardening/
│   │   └── tasks/main.yml
│   ├── user_management/
│   │   └── tasks/main.yml
│   ├── process_service_management/
│   │   └── tasks/main.yml
│   ├── storage_management/
│   │   └── tasks/main.yml
│   ├── server_tools/
│   │   └── tasks/main.yml
│   ├── game_center_policy/
│   │   └── tasks/main.yml
│   └── lab_academico_policy/
│       └── tasks/main.yml
│
├── playbook.yml                    # Orquestador principal
└── README.md                       # Documento técnico (este archivo)
```

---

### 🖥️ **Inventario y Topología**

El inventario (`inventory/hosts.yml`) define todos los equipos y su agrupación:

```yaml
all:
  children:
    lab_academico:
      hosts:
        server_academico:
          ansible_host: "2001:DB8:1:2::10"
        pc_academico_01:
          ansible_host: "2001:DB8:1:2::11"
        pc_academico_02:
          ansible_host: "2001:DB8:1:2::12"

    game_center:
      hosts:
        server_gamer:
          ansible_host: "2001:DB8:1:1::10"
        pc_win11:
          ansible_host: "2001:DB8:1:1::11"
        pc_ubuntu:
          ansible_host: "2001:DB8:1:1::12"
        pc_macos:
          ansible_host: "2001:DB8:1:1::13"
```

Los servidores (`server_academico`, `server_gamer`) poseen permisos administrativos para monitorear y limitar el uso de los equipos.

---

### 🧩 **Roles Principales**

| Rol                            | Descripción                                                               |
| ------------------------------ | ------------------------------------------------------------------------- |
| **common**                     | Configuraciones comunes (NTP, cron básico, actualizaciones).              |
| **security_hardening**         | Políticas de seguridad: desactivar root SSH, servicios innecesarios, etc. |
| **user_management**            | Creación de usuarios según el tipo de salón (`estudiante` o `gamer`).     |
| **process_service_management** | Control de procesos y servicios del sistema (top, ps, systemctl, etc.).   |
| **storage_management**         | Gestión de almacenamiento, revisión de espacio, limpieza de temporales.   |
| **server_tools**               | Instalación de herramientas de monitoreo como **Veyon** en servidores.    |
| **lab_academico_policy**       | Instalación de software académico permitido (7zip, VLC, VSCode).          |
| **game_center_policy**         | Instalación de software de entretenimiento (Steam, Discord, etc.).        |

---

### 🧰 **Variables Definidas**

Ejemplo: `group_vars/lab_academico.yml`

```yaml
---
nombre_usuario_salon: "estudiante"
politica_navegador: "restringido"
software_permitido:
  - 7zip
  - vlc
  - vscode
```

Ejemplo: `group_vars/game_center.yml`

```yaml
---
nombre_usuario_salon: "gamer"
politica_navegador: "abierto"
software_permitido:
  - steam
  - discord
  - 7zip
  - vlc
```

---

### 🔄 **Automatización de Tareas (cron y programador de tareas)**

Cada estación crea tareas automáticas para limpieza semanal de archivos temporales y mantenimiento básico.

* En **Linux**: se usa el módulo `cron`
* En **Windows**: se usa `win_scheduled_task`

---

### 🧩 **Gestión de Procesos y Servicios**

El rol `process_service_management` permite:

* Asegurar que servicios críticos estén activos (SSH, cron, LanmanWorkstation).
* Listar procesos en ejecución.
* Reiniciar o detener servicios según necesidad.

---

### 💾 **Gestión de Almacenamiento**

El rol `storage_management`:

* Muestra uso de disco.
* Limpia temporales en Linux y Windows.
* Permite extender almacenamiento en versiones futuras con `mount` o `win_disk_facts`.

---

### 🔐 **Gestión de Seguridad y Permisos**

El rol `security_hardening` aplica configuraciones para:

* Deshabilitar el acceso root por SSH.
* Bloquear servicios innecesarios.
* Fortalecer contraseñas por defecto.

Los roles `user_management` y `server_tools` complementan esta capa creando usuarios y configurando permisos diferenciados.

---

### 🚀 **Ejecución del Proyecto**

#### 🔹 Prerrequisitos

* Ansible instalado (`ansible --version`)
* Conectividad WinRM configurada para equipos Windows.
* Claves SSH o contraseñas válidas en las máquinas Linux/macOS.
* Inventario actualizado con direcciones correctas IPv6.

#### 🔹 Comando de ejecución

```bash
ansible-playbook playbook.yml
```

También puedes ejecutar por secciones:

```bash
ansible-playbook playbook.yml --tags "user_management"
ansible-playbook playbook.yml --limit game_center
```

---

### 📊 **Resultados Esperados**

| Área                     | Resultado                                                                                 |
| ------------------------ | ----------------------------------------------------------------------------------------- |
| **Usuarios y permisos**  | Creación automática de usuarios `estudiante` y `gamer` con sus permisos correspondientes. |
| **Procesos y servicios** | Servicios críticos activos, monitoreo funcional.                                          |
| **Tareas programadas**   | Limpieza semanal automática de temporales.                                                |
| **Almacenamiento**       | Optimización de espacio y monitoreo de uso.                                               |
| **Software instalado**   | Steam y Discord en Game Center, VSCode y VLC en Laboratorio Académico.                    |
| **Seguridad**            | SSH reforzado, servicios innecesarios deshabilitados.                                     |

---
