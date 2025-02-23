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
![ANSIBLE_ping](https://github.com/user-attachments/assets/1553ac21-a06b-4563-8a2e-fc2bf8101984)

Obtención del tiempo de actividad (todos los hosts):
![ANSIBLE_adhoc_uptime](https://github.com/user-attachments/assets/c5df853d-ce01-477c-9bbc-a45a3f0e85b7)

Instalación de apache (en webserver Centos):
![ANSIBLE_adhoc_apache](https://github.com/user-attachments/assets/335b0558-a1c3-42fe-a325-15c1d6de69a5)

Uso de espacio en disco (en servidor Ubuntu):
![ANSIBLE_adhoc_espacio](https://github.com/user-attachments/assets/584ede2d-50ad-425f-8318-e9f92789eb82)

Verificación de sintaxis playbook "web_setup.yml", ejecución y resultado (en webserver Centos): 
![ANSIBLE_webserver_playbook_sintaxis](https://github.com/user-attachments/assets/8d2a6eaa-b73b-49aa-8b88-b915c6e40994)
![ANSIBLE_webserver_playbook_ejecucion](https://github.com/user-attachments/assets/a6f63c5b-08e5-4f67-b176-2b876f48f25d)
![ANSIBLE_webserver_playbook_resultado2](https://github.com/user-attachments/assets/e469218c-1105-43dc-81fd-b021f1e99750)
![ANSIBLE_webserver_playbook_resultado3](https://github.com/user-attachments/assets/209e9607-1253-47e8-8da6-0bd30b8f9860)
![ANSIBLE_webserver_playbook_resultado](https://github.com/user-attachments/assets/75655cc9-9b55-45c7-91b6-2b70903c4590)

Verificación de sintaxis playbook "hardening.yml", ejecución y resultados (en servidor Ubuntu): 
![ANSIBLE_hardening_playbook_sintaxis](https://github.com/user-attachments/assets/4f3a3dc9-1ec4-4770-8f09-81a064facbf2)
Comando: ansible-playbook -i inventory.ini hardening.yml --become --ask-become-pass
![ANSIBLE_harening_playbook_ejecucion](https://github.com/user-attachments/assets/3d6627cf-bfa5-45ac-a803-6b0f5bd071f4)

Archivo visualizado: "/etc/ssh/sshd/sshd_config"
![ANSIBLE_hardening_playbook_resultado1](https://github.com/user-attachments/assets/4cbe12b9-6814-4b03-bf79-b6b220286fc4)
![ANSIBLE_hardening_playbook_resultado2](https://github.com/user-attachments/assets/df6e414b-7929-41e6-9909-c42861068209)

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
   
 ![ANSIBLE_hardening_error_verificar](https://github.com/user-attachments/assets/b32b9fff-147c-4a9a-b432-1e3810f45916)
 
 Otro error contrado fue que debíamos generar las claves para el usuario "sysadmin" y no para el usuario root (estaba especificado become:true pero no se especificó un usuario particular), esto se solucionó adicionando "become_user = sysadmin" cuando fuera necesario para generarle sus claves SSH.


* Problema: El host bastión no responde al módulo PING de ANSIBLE
![ANSIBLE_error_ping_bastion](https://github.com/user-attachments/assets/93d12380-f5c8-409b-9bb9-b7c19f56751a)

  * **Solución encontrada**: Para que el host Bastión respondiera el ping enviado por ANSIBLE, encontramos que podíamos copiar la clave SSH en sí mismo mediante el comando "ssh-copy-id sysadmi@$IP"
 
* Problema: El módulo ping de ANSIBLE falla por la carencia del paquete sshpass.
![ANSIBLE_error_sshpass](https://github.com/user-attachments/assets/a94e185f-bf3d-4bf6-8359-8ac37760d67a)
    
  * **Solución encontrada**: Encontramos que el error se resolvía simplemente instalando el paquete "sshpass" m mediante "dnf install sshpass -y" 
 

#

# Referencias

* Material del curso
* Documentación propia de ANSIBLE para cada Collection: https://docs.ansible.com/
* ChatGPT

Prompt:
"En un playbook de ansible, hay alguna forma de conectarse a un equipo remoto si aun no tengo definida una clave pública SSH para conectarme? le puedo indicar una contraseña? como?"

Prompt 2:
"Si en un playbook de ansible necesito crear una clave SSH pública, pero tengo la instrucción "become=true", necesariamente dicha llave se creará para el usuario root?"
