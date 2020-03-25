
# Ansible (Parte II)

## Ansible Roles
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

La web: https://galaxy.ansible.com/ es un repositorio de roles, puede contribuir con la comunidad compartiendo tus roles y usando los que allí se comparten.

## Modo comprobación (Dry Run)

Si se ejecuta ansible-playbook con la opción --check, no hará ningún cambio en los sistemas remotos, pero los módulos (que admitan el "modo de verificación") informará qué cambios llevaría a cabo. Los módulos que no admiten el modo de verificación no realizarán ninguna acción.

Ejemplo:

```
ansible-playbook foo.yml --check
```
Cada tarea del Playbook puede incluir el parámetro 'check_mode', si su valor es 'yes' (check_mode: yes) la tarea siempre se ejecutará en modo comprobación, aunque no se indique la opción --check, y si si valor es 'no' (check_mode: no) la tarea nunca se ejecutará en modo comprobación, aunque se indique la opción --check.

Por ejemplo:

```
tasks:
  - name: this task will make changes to the system even in check mode
    command: /something/to/run --even-in-check-mode
    check_mode: no

  - name: this task will always run under checkmode and not change the system
    lineinfile:
        line: "important config"
        dest: /path/to/myconfig.conf
        state: present
    check_mode: yes
```

Ansible dispone de una variable 'ansible_check_mode' que toma el valor 'True' cuando el Playbook es invocado con la opción --check, y 'False' en caso contrario.

En el siguiente ejemplo, la primera tarea se ejecuta en modo comprobación, mientras que la segunda tarea ignora los errores cuando se ejecuta en modo comprobación:

```
tasks:

  - name: this task will be skipped in check mode
    git:
      repo: ssh://git@github.com/mylogin/hello.git
      dest: /home/mylogin/hello
    when: not ansible_check_mode

  - name: this task will ignore errors in check mode
    git:
      repo: ssh://git@github.com/mylogin/hello.git
      dest: /home/mylogin/hello
    ignore_errors: "{{ ansible_check_mode }}"
```
## Gestión de errores

Ansible verifica los valores de retorno de los módulos y trata los errores de forma predeterminada, a menos que establezcamos lo contrario y cambiemos el comportamiento por defecto de Ansible para que el comportamiento ante un error o un valor de retorno de un módulo sea el que deseamos.


```
- hosts: webservers
  tasks:
  - name: Check status of apache
    command: service httpd status
    changed_when: false

  - name: this will not be counted as a failure
    command: /bin/false
    ignore_errors: yes
```

## Tags

Si tiene un playbook muy grande, puede ser útil poder ejecutar solo una parte específica . Ansible dispone un atributo "tags: <etiqueta>" por asociarlo a las tareas y los flags "--tags <etiqueta>" o "--skip-tags <etiqueta>" para especificarlos en la ejecución deplaybooks para que una tarea se lleve a cabo o no, respectivamente.

En el siguiente ejemplo, se asigna la etiqueta "packages" a la primera tarea y "configuration" a la segunda tarea: 

```
tasks:
- yum:
    name:
    - httpd
    - memcached
    state: present
  tags:
  - packages

- template:
    src: templates/src.j2
    dest: /etc/foo.conf
  tags:
  - configuration
```
De esta forma, el siguiente comando lleva a cabo ambas tareas:

```
ansible-playbook example.yml --tags "configuration,packages"
```

El siguiente, no lleva a cabo aquellas con la etiqueta "packages":

```
ansible-playbook example.yml --skip-tags "packages"
```

Se pueden listar las tareas que se ejecutarían bajo una etiqueta incluyendo la opción --list-tasks.

```
ansible-playbook example.yml --tags "configuration,packages" --list-tasks
```

## Encriptación de datos (Vault)


Ansible Vault es una característica que permite mantener en, archivos cifrados, los datos confidenciales como contraseñas o claves. Estos archivos se pueden distribuir o colocarlos en un sistema de control de versiones.

El comando "ansible-vault" nos permite crear un fichero encriptado asignandole una password. El flag --vault-id asigna un id específico para el fichero y solicita las password.

```
ansible-vault create --vault-id vault1@prompt secret.yml
```

Para ejecuatr un playbook, el flag --ask-vault-pass solicita la password cuando el playbook incluye un fichero encriptado.

```
ansible-playbook --ask-vault-pass example.yml
```

El flag --vault-password-file establece el fichero con la password

```
ansible-playbook --vault-password-file /path/to/password-file example.yml
```

El flag --vault-id permite indicar el id y la password en la línea de comando:

```
ansible-playbook --vault-id  vaultid@password example.yml
```

Para editar el contenido de un fichero encriptado:

```
ansible-vault edit secret.yml
```

Para ver el contenido de un fichero encriptado:

```
ansible-vault view secret.yml
```

Para desenciprtar un fichero:

```
ansible-vault decrypt secret.yml
```

## Prompt

Los Prompts permiten definir valores que el usuario introducirá cuando se ejecute el Playbook, por ejemplo datos confidenciales, etc. Para ello, se debe incluir el elemento "vars_prompt".

Aquí hay un ejemplo básico:

```
# prompt.yml
---
- hosts: all
  vars_prompt:
    - name: username
      prompt: "What is your username?"
      private: no

    - name: password
      prompt: "What is your password?"

  tasks:

    - debug:
        msg: 'Logging in as {{ username }}'
```


## Condiciones con when

A veces será necesario realizar una tarea o no dependiendo de ciertas condiciones, por ejemplo, instalar o no un paquete dependiendo del sistema operativo o eliminar ficheros si el disco se está llenando.

Para ello, Ansible dispone de la cláusula when, que utiliza una expresión Jinja2 (sin llaves dobles). Algunos ejemplos:

```
tasks:
  - name: "shut down Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_facts['os_family'] == "Debian"
```

Para agrupar condiciones se utilizan los paréntesis:
```
tasks:
  - name: "shut down CentOS 6 and Debian 7 systems"
    command: /sbin/shutdown -t now
    when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
          (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")
```
Si indicamos varias condiciones, estarían unidas mediante un 'Y' lógico:

```
tasks:
  - name: "shut down CentOS 6 systems"
    command: /sbin/shutdown -t now
    when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"
```

Un ejemplo más completo:

```
# setup-app2.yml

---
  - hosts: webservers
    become: true

    vars:
      path_to_app: "/var/www/html"

    vars_prompt:
      - name: "upload_var"
        prompt: "Upload the index.php file?"
      - name: "create_var"
        prompt: "Create info.php page?"
 
    tasks:
      - name: Upload application file
        copy:
          src: ../index.php
          dest: "{{ path_to_app }}"
          mode: 0755
        when: upload_var == 'yes'
        tags: upload

      - name: Create simple info page
        copy:
          dest: "{{ path_to_app }}/info.php"
          content: "<h1> Info about our webserver {{ ansible_hostname }}. These are changes. </h1>"
        when: create_var == 'yes'
        tags: create

      - name: Configure php.ini file
        lineinfile:
          path: /etc/php.ini
          regexp: ^short_open_tag
          line: 'short_open_tag=On'
        notify: restart apache

    handlers:
      - name: restart apache
        service: name=httpd state=restarted
```



---
JLAM, 2020. 