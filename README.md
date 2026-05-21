# 🏫 Ansible - Arriagakoharitza Ikastetxea
## Gestión de clientes Linux Mint Cinnamon

---

## 📋 Estructura del proyecto

```
school-ansible/
├── ansible.cfg                     # Configuración de Ansible
├── site.yml                        # Playbook principal (entry point)
├── inventory/
│   ├── hosts.ini                   # Inventario de equipos
│   └── group_vars/
│       ├── all.yml                 # Variables globales
│       ├── profesores.yml          # Variables perfil profesores
│       ├── alumnos.yml             # Variables perfil alumnos
│       ├── administracion.yml      # Variables perfil admin
│       └── vault.yml               # Secretos (cifrado con ansible-vault)
├── playbooks/
│   ├── 01_base_setup.yml           # Sistema base, locale, NTP, DNS
│   ├── 02_freeipa_client.yml       # Unión al dominio FreeIPA
│   ├── 03_software_install.yml     # Software educativo
│   ├── 04_firefox_config.yml       # Personalización Firefox
│   ├── 05_cups_client.yml          # Configuración impresoras
│   ├── 06_glpi_agent.yml           # Agente de inventario GLPI
│   ├── 07_security.yml             # Seguridad y bastionado
│   ├── 08_desktop_config.yml       # Escritorio Cinnamon por perfil
│   ├── 09_network_shares.yml       # Recursos de red NFS
│   └── 10_maintenance.yml          # Mantenimiento y actualizaciones
├── templates/
│   └── chrony.conf.j2              # Plantilla NTP
└── files/
    ├── wallpapers/                  # Fondos de pantalla corporativos
    └── firefox/                     # Ficheros adicionales Firefox
```

---

## 🚀 Primeros pasos

### 1. Requisitos en el servidor Ansible
```bash
# En ans.arriagakoharitza.net
sudo apt install ansible python3-pip

# Colecciones necesarias
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

### 2. Generar clave SSH para Ansible
```bash
ssh-keygen -t ed25519 -f ~/.ssh/ansible_ed25519 -C "ansible@arriagakoharitza.net"

# Distribuir la clave a los clientes (primera vez, manualmente o con PXE/cloud-init)
ssh-copy-id -i ~/.ssh/ansible_ed25519.pub ansible_admin@<IP_CLIENTE>
```

### 3. Configurar el vault con contraseñas
```bash
cp inventory/group_vars/vault.yml.example inventory/group_vars/vault.yml
ansible-vault encrypt inventory/group_vars/vault.yml
# Introduce una contraseña segura para el vault
```

### 4. Editar el inventario
```bash
nano inventory/hosts.ini
# Añadir los equipos con sus IPs reales
```

### 5. Verificar conectividad
```bash
ansible clientes -m ping
ansible all -m setup --limit pc-prof-01.arriagakoharitza.net
```

---

## 🎮 Comandos de uso habitual

### Despliegue completo (equipo nuevo)
```bash
ansible-playbook site.yml --ask-vault-pass
```

### Sólo un perfil
```bash
ansible-playbook site.yml --limit profesores --ask-vault-pass
ansible-playbook site.yml --limit alumnos --ask-vault-pass
ansible-playbook site.yml --limit administracion --ask-vault-pass
```

### Sólo un equipo
```bash
ansible-playbook site.yml --limit pc-prof-01.arriagakoharitza.net --ask-vault-pass
```

### Sólo algunas tareas (tags)
```bash
# Instalar/actualizar solo software
ansible-playbook site.yml --tags software --ask-vault-pass

# Solo Firefox
ansible-playbook site.yml --tags firefox --ask-vault-pass

# Solo seguridad
ansible-playbook site.yml --tags seguridad --ask-vault-pass

# Mantenimiento (actualizaciones)
ansible-playbook site.yml --tags mantenimiento --ask-vault-pass

# Escritorio Cinnamon
ansible-playbook site.yml --tags escritorio --ask-vault-pass
```

### Modo dry-run (ver cambios sin aplicar)
```bash
ansible-playbook site.yml --check --diff --ask-vault-pass
```

### Mantenimiento semanal (fuera de horario)
```bash
# Programar en cron del servidor Ansible (domingos a las 22:00)
# crontab -e
# 0 22 * * 0 ansible-playbook /home/ansible/school-ansible/site.yml --tags mantenimiento --vault-password-file ~/.vault_pass >> /var/log/ansible/cron.log 2>&1
```

---

## 🖥️ Infraestructura

| Servidor | IP | FQDN | Función |
|---|---|---|---|
| FreeIPA | 10.2.101.231 | ipa.arriagakoharitza.net | Directorio, DNS, NTP, Kerberos |
| CUPS | 10.2.101.232 | cups.arriagakoharitza.net | Impresión |
| Ansible | 10.2.101.233 | ans.arriagakoharitza.net | Gestión de configuración |
| GLPI | 10.2.101.234 | glpi.arriagakoharitza.net | Inventario y tickets |

---

## 🔖 Tags disponibles

| Tag | Playbook | Descripción |
|---|---|---|
| `base` | 01 | Configuración base del sistema |
| `dominio` / `ipa` | 02 | Inscripción FreeIPA |
| `software` / `paquetes` | 03 | Instalación de software |
| `firefox` / `navegador` | 04 | Configuración Firefox |
| `cups` / `impresoras` | 05 | Impresoras |
| `glpi` / `inventario` | 06 | Agente GLPI |
| `seguridad` / `hardening` | 07 | Bastionado |
| `escritorio` / `cinnamon` | 08 | Escritorio por perfil |
| `nfs` / `shares` | 09 | Recursos de red |
| `mantenimiento` | 10 | Actualizaciones y limpieza |

---

## 📦 Playbooks adicionales sugeridos

Además de los incluidos, considera añadir:

- `11_aula_modo_examen.yml` — Bloquear internet y apps durante exámenes
- `12_recuperacion_home.yml` — Restaurar perfil de usuario corrupto
- `13_despliegue_imagen.yml` — Clonar configuración base a equipos nuevos
- `14_recoleccion_logs.yml` — Centralizar logs en servidor (rsyslog/ELK)
- `15_actualizacion_firefox.yml` — Actualizar políticas Firefox sin reboot

---

## ⚠️ Notas importantes

1. **Vault**: nunca subas `vault.yml` a un repositorio sin cifrar
2. **FreeIPA enrollment**: requiere conectividad con el servidor IPA antes de ejecutar el playbook 02
3. **Mantenimiento**: el playbook 10 solo reinicia equipos fuera del horario escolar (20:00–06:00)
4. **Fondos de pantalla**: coloca tu imagen en `files/wallpapers/arriagakoharitza_fondo.jpg` antes de ejecutar el playbook 08
5. **NFS**: ajusta las rutas de `shares_nfs` en los group_vars según tu servidor NAS real
