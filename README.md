# Obligatorio: Taller de Linux 2025
## Documento informativo
#

Autores:
- ###### Pablo Delucchi - N° 315123 
- ###### Agustín Guarteche - N° 240159

&nbsp;

#### Requerimientos previos:
###### En el directorio "collections" del repositorio se encuentra el archivo "requirements.yml" con los nombres de las colecciones de ANSIBLE necesarias para ejecutar los playbook correctamente. Estas, bien pueden instalarse manualmente mediante ansible galaxy:

```bash
ansible-galaxy ansible.posix install 
ansible-galaxy community.general install
ansible-galaxy community.crypto install
```

###### o bien en forma automática invocando el archivo de definisiones desde la raíz:

```bash
ansible-galaxy install -r collections/requirements.yml
```
#

&nbsp;

# Pruebas de ejecución 

Módulo PING (todos los hosts):
![ANSIBLE_IMG1](file:results/ANSIBLE_ping.JPG)

Obtención del tiempo de actividad (todos los hosts):
![ANSIBLE_IMG2](file:results/ANSIBLE_adhoc_uptime.JPG)

Instalación de apache (en webserver Centos):
![ANSIBLE_IMG3](file:results/ANSIBLE_adhoc_apache.JPG)

Uso de espacio en disco (en servidor Ubuntu):
![ANSIBLE_IMG4](file:results/ANSIBLE_adhoc_espacio.JPG)

Verificación de sintaxis playbook "web_setup.yml", ejecución y resultado (en webserver Centos): 
![ANSIBLE_IMG5](file:results/ANSIBLE_webserver_playbook_sintaxis.JPG)
![ANSIBLE_IMG6](file:results/ANSIBLE_webserver_playbook_ejecucion.JPG)
![ANSIBLE_IMG7](file:results/ANSIBLE_webserver_playbook_resultado2.JPG)
![ANSIBLE_IMG8](file:results/ANSIBLE_webserver_playbook_resultado3.JPG)
![ANSIBLE_IMG9](file:results/ANSIBLE_webserver_playbook_resultado.JPG)

Verificación de sintaxis playbook "hardening.yml", ejecución y resultados (en servidor Ubuntu): 
![ANSIBLE_IMG10](file:results/ANSIBLE_hardening_playbook_sintaxis.JPG)
Comando: ansible-playbook -i inventory.ini hardening.yml --become --ask-become-pass
![ANSIBLE_IMG11](file:results/ANSIBLE_harening_playbook_ejecucion.JPG) 

Archivo visualizado: "/etc/ssh/sshd/sshd_config"
![ANSIBLE_IMG12](file:results/ANSIBLE_hardening_playbook_resultado1.JPG) 
![ANSIBLE_IMG13](file:results/ANSIBLE_hardening_playbook_resultado2.JPG) 

Obs: Como ya se encontraba habilitado el tráfico por el puerto por defecto de SSH, procedimos a agregarlo nuevamente (sin especificar protocolo para que pudiera verse el cambio generado)


# Desafíos/problemas encontrados
* Problema: IP dinámica de "Bastión".
    * Durante el taller establecimos como fijas las IP de los host 
      "Ubuntu-srv" y "Centos-SRV", pero a bastión no le configuramos una IP, por lo 
      que esta es obtenida por DHCP. Debido a ello esta puede cambiar según que equipo ejecute el playbook y hacer que falle.

     * **Solución encontrada**: Establecimos una variable "bastion_ip" en el 
     archivo "inventory.ini" y la referenciamos en el grupo. 

* Problema: Caso borde clave pública SSH inexistente al conectar.
    * Se solicitaba por letra que nos aseguráramos de que la clave 
      pública de bastión se encontraba en el servidor de Ubuntu. 
      Puede darse el caso de borde de que el archivo "know_host" no 
      exista en el servidor o que no contenga la clave pública de 
      bastión, en cuyo caso ANSIBLE no es capaz de establecer conexión 
      con el host.
      
    * **Solución encontrada**: Se establece la variable "ansible_ssh_pass" en el archivo de inventario para una permitir una primera conexión al servidor por SSH con contraseña, de esta manera, si el archivo no existe o la clave no está presente, de igual manera el playbook se ejecuta y, llegado el caso, crea el archivo o añade la clave. Posteriormente se deshabilita el 
    acceso mediante contraseña, pero al haberse verificado la existencia de la clave, esto ya no es un problema.  

* Problema: Error de ejecución en playbook en caso de no existir archivo de host conocidos. Creación de clave propietaria

  * **Solución encontrada**: Durante la ejecución del playbook se verifica si hay alguna clave SSH para el usuario "sysadmin" en el archivo de host conocidos. (el cual previamente se crea si no existe) El problema que estábamos teniendo era que si la clave no existe en el archivo (es decir si la salida era vacía) nos generaba un error que detenía el playbook. como solución encontramos que podíamos agregar una instrucción "ignore_errors: true" para que en dicho caso continuara la ejecución.
 
 ![ANSIBLE_IMG14](file:results/ANSIBLE_hardening_error_verificar.JPG) 
 
 Otro error contrado fue que debíamos generar las claves para el usuario "sysadmin" y no para el usuario root (estaba especificado become:true pero no se especificó un usuario particular), esto se solucionó adicionando "become_user = sysadmin" cuando fuera necesario para generarle sus claves SSH.

#

# Referencias

* Material del curso
* Documentación propia de ANSIBLE para cada Collection: https://docs.ansible.com/
* ChatGPT

Prompt:
"En un playbook de ansible, hay alguna forma de conectarse a un equipo remoto si aun no tengo definida una clave pública SSH para conectarme? le puedo indicar una contraseña? como?"

Prompt 2:
"Si en un playbook de ansible necesito crear una clave SSH pública, pero tengo la instrucción "become=true", necesariamente dicha llave se creará para el usuario root?"
