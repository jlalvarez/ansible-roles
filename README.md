
# Ansible Roles
https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html

Los roles permiten cargar automáticamente ciertos vars_files, tareas, handlers, etc. y este agrupamiento por roles permite compartirlos con otros usuarios.

Genera una estructura:

```
site.yml
webservers.yml
fooservers.yml
roles/
    webservers/
        tasks/
          - main.yml
        vars/
          - main.yml
        handlers/
          - main.yml
    common/
        tasks/
        handlers/
        files/
        templates/
        vars/
        defaults/
        meta/
```

Los roles se usarían en la Playbooks con:

```
---
- hosts: webservers
  roles:
    - common
    - webservers
```
El comportamiento para cada rol "x" sería:

- Si existe roles/x/task/main.yml, las tareas incluidas en el main.yml se agregarán al Play.
- Si existe roles/x/handlers/main.yml, los handlers serán agregados al play.
- Si existe roles/x/vars/main.yml, se agregarán las variables.
- Si existe roles/x/defaults/main.yml, se agregarán las variables.
- Si existe roles/x/meta/main.yml, se agregarán las dependencias de roles.
- Cualquier copia, script, plantilla o tareas incluidas (en el rol) pueden hacer referencia a archivos incluidos es roles/x/{archivos, plantillas, tareas}/.

### Comando Ansible Galaxy 

Ansible Galaxy es un sitio gratuito para buscar, descargar, calificar y revisar todo tipo de roles de Ansible desarrollados por la comunidad y puede ser una excelente manera de comenzar sus proyectos de automatización.

El comando ansible-galaxy está incluido en Ansible. El cliente Galaxy le permite descargar roles de Ansible Galaxy, y también proporciona un excelente marco predeterminado para crear sus propios roles.

Podemos crear un rol con:

```
$ ansible-galaxy roles/webservers init
```
Insertaremos en el main.yml del elemento concreto el contenido deseado.

