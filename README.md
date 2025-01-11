# A playground to test Ansible with Docker Compose & Ubuntu

## Actions in the Host machine:

Launch containers
```bash
docker-compose up -d
```

Jump to the control node from host machine
```bash
docker-compose exec control bash
```

Cleanup
```bash
docker compose down
```
![image](https://github.com/user-attachments/assets/9c35f805-6e87-41aa-8983-13ce9f031544)
![image](https://github.com/user-attachments/assets/094ee43d-67ea-4ecb-b208-9d9e5ef23023)


# Inside control node

Install ansible
```bash
sudo apt update
sudo apt-get install nano
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
```

![image](https://github.com/user-attachments/assets/f5d3ae0f-0e72-4dc8-ac15-a2b95a014354)
![image](https://github.com/user-attachments/assets/db3d168e-5966-496b-bae3-15071b1841a7)


Setup ansible
```bash
sudo nano /etc/ansible/hosts
```

Content of host the file:
```
[servers]
server1 ansible_host=node1
server2 ansible_host=node2
server3 ansible_host=node3

[all:vars]
[all:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=password
ansible_python_interpreter=/usr/bin/python3
```
![image](https://github.com/user-attachments/assets/0cd925cb-720f-4956-b5d4-356f5e7507a7)

Test conectivity
```bash
ansible-inventory --list -y
ansible all -m ping -u root
ansible all -a "df -h" -u root
```

![image](https://github.com/user-attachments/assets/a017471e-8146-4afc-927b-eaf96ca57d90)

Connect with nodes (ssh password is "password" without quotes):
```bash
ssh root@node1
ssh root@node2
ssh root@node3
```
![image](https://github.com/user-attachments/assets/4860dda3-e7f8-4da4-b06a-678e9cdbd0ba)
![image](https://github.com/user-attachments/assets/875ccf75-a026-447a-8ce1-2eef7f7cd42d)
![image](https://github.com/user-attachments/assets/d1ba6336-dde8-4490-bb10-48eed70d5fc3)


Shared folder between host machine and control node:
```bash
cd /shared
```

And do the magic here!

![image](https://github.com/user-attachments/assets/30820b89-698e-405f-9f97-e46dfacf069e)
![image](https://github.com/user-attachments/assets/455800b4-554f-4bfa-a2a5-59625f5fd493)

Creamos una nueva receta:

```
---
- name: Configure open ports and services
  hosts: all
  become: yes

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
      update_cache: yes

  - name: Start and enable nginx service
    systemd:
      name: nginx
      state: started
      enabled: yes

  - name: Open port 80 using UFW
    ansible.builtin.ufw:
      rule: allow
      port: 80
      proto: tcp

  - name: Verify nginx is running
    ansible.builtin.shell: curl -I http://localhost
    register: nginx_status

  - name: Debug nginx response
    debug:
      var: nginx_status.stdout

```
![image](https://github.com/user-attachments/assets/9386c0e3-f6d3-4789-90d1-033f1022c2d7)

## Script para Configuración Automática

Escribimoa un script en Bash que ejecute el playbook y capture la salida. Creamos un archivo llamado run_playbook.sh

```
#!/bin/bash
# Script para ejecutar un playbook de Ansible

# Variables
PLAYBOOK="shared/recipe-ports.yml"
LOG_FILE="playbook_output.log"

# Ejecutar el playbook
echo "Ejecutando playbook: $PLAYBOOK"
ansible-playbook $PLAYBOOK | tee $LOG_FILE

# Verificar el resultado
if grep -q 'failed=0' $LOG_FILE; then
  echo "El playbook se ejecutó correctamente."
else
  echo "Hubo errores durante la ejecución del playbook. Verifica $LOG_FILE para más detalles."
fi

```

![image](https://github.com/user-attachments/assets/a323ff68-204a-4104-b59e-70e1dd02e8d4)

### Ejecutamos el script
```
/shared/run_playbook.sh
```
![image](https://github.com/user-attachments/assets/70e49d35-0bf6-4074-b08f-44e6ba249a98)
![image](https://github.com/user-attachments/assets/4fc99724-a4ec-4b86-9648-15a0d9fc0258)

