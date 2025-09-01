# 🚀 Proyecto InnovaSys – Automatización con Ansible

Este repositorio contiene un proyecto de **aprovisionamiento automatizado** usando **Ansible** para la startup _InnovaSys_.  
El servidor objetivo (Ubuntu Server 24.04) será configurado con:

1. 🌐 **Portal de Bienvenida (Intranet)** → Servidor web Apache con página personalizada.
2. 📂 **Repositorio de Archivos Centralizado** → Servidor Samba con recurso compartido seguro para el equipo de desarrollo.

---

## 📂 Estructura del Proyecto

```
InnovaSys-Ansible/
├─ hotsts # Inventario Ansible con IPs y usuarios
├─ site.yml # Playbook principal
├─ group_vars/
│ └─ servidor-innovasys.yml # Variables específicas del host
├─ ansible.cfg # Configuración de Ansible
├─ roles/
│ ├─ apache/
│ │ ├─ tasks/main.yml
│ │ ├─ handlers/main.yml
│ │ ├─ templates/index.html.j2
│ │ └─ defaults/main.yml
│ └─ samba/
│ ├─ tasks/main.yml
│ ├─ handlers/main.yml
│ ├─ templates/smb.conf.j2
│ └─ defaults/main.yml
```

---

## ⚙️ Requisitos

- **Nodo de control:** Linux Lite con Ansible instalado.
- **Nodo gestionado:** Ubuntu Server 24.04 virgen.
- **SSH sin contraseña** configurada desde el nodo de control al gestionado.
- Python en el nodo gestionado (generalmente ya instalado en Ubuntu Server).
- Conexión de red entre ambos nodos.

---

## 🔑 Configuración SSH sin contraseña

1. Generar par de claves en el nodo de control:

```bash
ssh-keygen -t rsa -b 4096
```

2. Copiar la clave pública al nodo gestionado:

```bash
ssh-copy-id operador@192.168.10.100
```

3. Probar conexión:

```bash
ssh operador@192.168.10.100
```

---

## 📌 Configuración de Ansible

### Archivo (`ansible.cfg`)

```ini
[defaults]
inventory = ./hosts
roles_path = ./roles
host_key_checking = True          # Recomendado en producción
retry_files_enabled = False
forks = 10
timeout = 30
interpreter_python = auto

bin_ansible_callbacks = True
result_format = yaml

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False
```

🔹 host_key_checking = True asegura que se valide la identidad del servidor remoto antes de ejecutar tareas.

### Inventario (`hosts`)

Inventario con el servidor gestionado:

```ini
[innovasys]
servidor-innovasys ansible_host=192.168.10.100 ansible_user=operador ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Variables globales (`group_vars/innovasys.yml`)

```yaml
nombre_empresa: "InnovaSys"
samba_share_name: "Proyectos"
samba_share_path: "/srv/samba/proyectos"
samba_group: "operadores"
samba_user: "operador1"
samba_password: "Innova.2025"
samba_valid_users: "@operadores"
samba_writable: "yes"
samba_browseable: "yes"
samba_create_mask: "0660"
samba_directory_mask: "2770"
```

🔒 La contraseña de Samba puede encriptarse con Ansible Vault para mayor seguridad.

---

## 🧩 Roles de Ansible

### 1️⃣ Rol: apache

- Instala Apache2
- Crea página de bienvenida personalizada desde plantilla Jinja2
- Inicia y habilita el servicio

### 2️⃣ Rol: samba

- Instala Samba
- Crea grupo y usuario del sistema (operadores, operador1)
- Crea directorio compartido /srv/samba/proyectos
- Configura Samba (smb.conf) con acceso solo a desarrolladores
- Establece contraseña Samba y reinicia el servicio si cambia la configuración

---

## 💻 Ejecución del Playbook

En el nodo de control, ejecutar:

```bash
ansible-playbook site.yml
```

Si tu usuario remoto necesita sudo con contraseña:

```bash
ansible-playbook site.yml -K
```

---

## ✅ Verificación

### 🌐 Portal Intranet

Abrir navegador en la máquina de control (Linux Lite):

```cpp
http://192.168.10.100/
```

Deberías ver:
“Bienvenidos a la Intranet de InnovaSys”

### 📂 Recurso compartido Samba

Desde el gestor de archivos:

```cpp
smb://192.168.10.100/Proyectos
```

- Usuario: operador1
- Contraseña: Innova.2025
- Deberías poder crear y editar archivos dentro del directorio compartido.

---
