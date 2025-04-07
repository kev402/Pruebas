# Descubriendo, interceptando y bloqueando un ReverseShell

## Recursos utilizados

- Script de Python para crear la conexión (clonado de [ReverseShell_Python](https://github.com/Maalfer/ReverseShell_Python))

- Equipo Windows 11 como víctima.

- Celular con Termux como atacante.

- Máquina virtual de Kali Linux Purple como defensor.

### Creación del programa 

El repositorio ya mencionado, muestra como hacer un Reverse shell sencillo en donde cada máquina tiene un archivo correspondiente (víctima.py y atacante.py), es sí es funcional y no es detectado por antivirus ya que como tal no es un payload o un comando malicioso, solo usa librerías comunes de Python, sin embargo levanta sospechas ya que el script está visiblemente ejecutándose en la consola además que el usuario víctima lo debe ejecutar manualmente, lo cual no es muy aplicable a un entorno de infección real.

Debido a esto quise hacerlo más real y simulando que también poseía una máquina Windows diferente a la víctima, modifiqué el código de víctima.py para encapsular todo el bucle en una función, luego cree una app inofensiva a simple vista (una calculadora de raíz cuadrada en tkinter) y desde ahí importé el archivo "malicioso" e hice que se activara después de que el usuario cierre la interfaz gráfica, así finalmente convertí el script en un archivo .exe usando pyinstaller con el parámetro --noconsole para que una vez se cierre el programa en segundo plano e invisiblemente se ejecute la conexión remota.

![víctima.py modificado](/victima.png)

![Calculadora "inofensiva"](/calculadora.png)

![Creación del ejecutable](/creacion.png)

### Ejecución de comandos 

En el celular desde Termux se debe tener el archivo atacante.py que apunta a la IP de la víctima, este se ejecuta y queda en escucha de cuando el usuario cierre la interfaz gráfica, cuando el script se ejecuta en segundo plano, el atacante puede empezar a ejecutar comandos:

![Comando ejecutado desde Termux](/whoami.jpg)

Incluso después de todo esto, el antivirus no detecta nada porque como ya dije, la librería socket y subprocess de Python en si no hacen nada malicioso, ya que no estamos usando comandos como base64 que hagan una conexión remota o enviando payloads de metasploit.

### Levantamiento de sospechas 

Aunque no hagamos ninguna alerta para los antivirus, la conexión que realizamos no está cifrada por lo que cualquier otra persona con conocimientos en ciberseguridad puede interceptar los comandos que ejecutamos y verlos en texto plano, además que dentro de la red se puede ver claramente que estamos abriendo el puerto 5000 de la víctima y esto se puede descubrir usando Nmap que viene en Kali Linux y la versión purple como se ve a continuación donde se realizó un escaneo general de toda la red usando el comando:

`nmap 192.168.1.1/24`

Esto mostró varias IP de la red, entre ellas la del equipo víctima:

![Resultado de escaneo](/scan.png)

Ahí es donde nos damos cuenta de que algo anda mal, y por ello es que se puede recurrir a la herramienta Wireshark para ver qué posibles paquetes están circulando por este puerto. Para unos resultados más limpios añadimos un filtro solo para el puerto 5000.

![Filtrar el puerto 5000](/puerto.png)

Descubriendo como pasa un comando por la red y es capturado en texto plano a la vez que se captura la respuesta que la víctima envía:

![Comando dir ejecutado desde el atacante y visualizado por Wireshark](/comando.png)

![Respuesta de la víctima](/respuesta.png)

Con esto se puede afirmar de que se trata de un reverseshell, para quitarle la conexión usé mi herramienta [GuardKev402](https://github.com/kev402/GuardKev402) (no es propaganda a la app, es de código abierto y no tiene fines lucrativos) ya que con ella en Windows pude buscar el puerto 5000 y el nombre de la app que lo ejecutaba (la calculadora en tkinter) cerrando así el proceso y cortando la conexión con el atacante:

![Encontrando el nombre del proceso, su ruta y el puerto abierto](/guardkev.png)
 
Finalmente escribí su nombre en la parte inferior, lo terminé y en Termux (el atacante) la conexión ya no respondía:

![Corte del socket en Termux](/final.jpg)

## IMPORTANTE!

Todo este procedimiento se hizo en un entorno controlado y en un red local propia, las herramientas usadas para el ataque son destinadas a lo ético y el aprendizaje, no soy responsable de cualquier uso incorrecto o no ético, etc de estas herramientas por parte de otras personas, todos los dispositivos y máquinas virtuales son de mi propiedad, en ningún momento se vulneraron dispositivos ajenos, el único ebjetivo de esto es educar y demostrar.