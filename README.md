
# Proyecto de Automatización con Ansible

Este proyecto automatiza la instalación y configuración del servidor Apache en varios nodos Linux (Rocky Linux), utilizando Ansible con una estructura organizada por ambientes y roles.

---

## 📌 Requisitos

- Rocky Linux (en nodo controlador y nodos gestionados)
- Ansible instalado en el nodo controlador
- Conectividad SSH sin contraseña (con claves públicas) entre el nodo controlador y los nodos gestionados
- El usuario `gio` debe tener permisos sudo en los nodos gestionados

---

## 🧱 Estructura del Proyecto

```bash
ansible/
├── inventories/
│   ├── development/
│   │   └── hosts
│   ├── staging/
│   │   └── hosts
│   └── production/
│       └── hosts
├── roles/
│   └── apache/
│       ├── tasks/
│       │   └── main.yml
│       └── templates/   # (opcional, si se usa Jinja2)
├── playbook-con-roles.yml
└── README.md
```

---

## 🧪 Parte 1: Configuración inicial

1. Actualización del sistema y configuración SSH:
   ```bash
   sudo dnf update -y
   sudo dnf install -y openssh-server
   sudo systemctl enable --now sshd
   ```

2. Generación de claves SSH:
   ```bash
   ssh-keygen
   ssh-copy-id gio@192.168.86.136
   ssh-copy-id gio@192.168.86.137
   ```

3. Verificación de conexión:
   ```bash
   ansible all -m ping -i inventories/development/hosts
   ```

4. Prueba con un Playbook básico para instalar Apache y crear una página HTML.

---

## ⚙️ Parte 2: Uso de Roles y Ambientes

### 🗂️ Inventario por ambiente

Archivo: `inventories/development/hosts`

```ini
[webservers]
192.168.86.136
192.168.86.137
```

### 📦 Rol: Apache

Archivo: `roles/apache/tasks/main.yml`

```yaml
---
- name: Instalar Apache
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

- name: Asegurar que Apache esté iniciado y habilitado
  become: true
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

- name: Crear archivo index.html
  become: true
  ansible.builtin.copy:
    dest: /var/www/html/index.html
    content: "<html><body><h1>Bienvenido a Ansible</h1></body></html>"
    mode: '0644'
```

### ▶️ Playbook: `playbook-con-roles.yml`

```yaml
---
- name: Desplegar Apache usando role
  hosts: webservers
  become: true
  roles:
    - apache
```

### ▶️ Ejecutar el playbook

```bash
ansible-playbook -i inventories/development/hosts playbook-con-roles.yml --ask-become-pass
```

---

## ✅ Resultado Esperado

- Apache instalado en ambos nodos (136 y 137)
- Página web en `http://192.168.86.136` y `http://192.168.86.137` mostrando:
  ```
  Bienvenido a Ansible
  ```

---

## 📚 Próximos pasos (Parte 3)

- Uso de plantillas Jinja2 (`templates/`)
- Variables por ambiente
- Handlers
- Notificaciones
- Más roles (por ejemplo, para MySQL, Nginx, etc.)

---

## 🧑‍💻 Autor

Automatizado por Giovana Segura 😊


# Proyecto de Automatización con Ansible y Kubernetes

Este proyecto tiene como objetivo automatizar la instalación de servicios en servidores Linux (CentOS 7), incluyendo Apache y Kubernetes, utilizando **Ansible** mediante **roles organizados y reutilizables**.

---

## 📁 Estructura del proyecto

```bash
ansible/
├── inventories/
│   └── development/
│       └── hosts
├── playbook-apache.yml
├── playbook-k8s.yml
└── roles/
    ├── apache/
    │   └── tasks/
    │       └── main.yml
    └── kubernetes/
        └── tasks/
            └── main.yml
```

---

## 🖥️ Inventario (`inventories/development/hosts`)

```ini
[web]
192.168.86.136
192.168.86.137
```

---

## 📜 Playbooks

### `playbook-apache.yml`

Instala Apache en los nodos definidos.

```yaml
- name: Instalar Apache en los nodos
  hosts: web
  become: true

  roles:
    - apache
```

### `playbook-k8s.yml`

Prepara los nodos para Kubernetes, instalando kubeadm, kubelet y kubectl, además de Docker.

```yaml
- name: Instalar Kubernetes en los nodos
  hosts: web
  become: true

  roles:
    - kubernetes
```

---

## 🧱 Rol: `apache`

Ubicación: `roles/apache/tasks/main.yml`

```yaml
- name: Instalar Apache
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

- name: Asegurar que Apache esté iniciado y habilitado
  become: true
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

- name: Crear archivo index.html
  become: true
  ansible.builtin.copy:
    dest: /var/www/html/index.html
    content: "<html><body><h1>Bienvenido a Ansible</h1></body></html>"
    mode: '0644'
```

---

## ☸️ Rol: `kubernetes`

Ubicación: `roles/kubernetes/tasks/main.yml`

- Desactiva `swap`.
- Instala dependencias del sistema.
- Configura los repos de Docker y Kubernetes.
- Instala Docker.
- Instala `kubeadm`, `kubelet` y `kubectl`.

> Asegúrate de tener acceso a internet en los nodos para que los repos funcionen correctamente.

---

## 🚀 Cómo ejecutar los playbooks

### 1. Apache:

```bash
ansible-playbook -i inventories/development/hosts playbook-apache.yml --ask-become-pass
```

### 2. Kubernetes:

```bash
ansible-playbook -i inventories/development/hosts playbook-k8s.yml --ask-become-pass
```

---

## ✍️ Autor

Proyecto guiado y documentado por Giovana Segura 
