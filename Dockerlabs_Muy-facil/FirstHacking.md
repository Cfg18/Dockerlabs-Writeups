# Writeup: FirstHackingCTF

* **Autor:** Cfg18
* **Plataforma:** DockerLabs
* **Dificultad:** Muy Fácil
* **SO:** Linux
* **Categoría:** Linux
* **Fecha:** 25/08/2026

## Resumen Ejecutivo (TL;DR)
Se realizó la auditoría de seguridad sobre la máquina objetivo identificando el servicio FTP activo. Mediante el análisis de la versión del servicio, se detectó una vulnerabilidad conocida como CVE-2011-2523 (puerta trasera en vsftpd 2.3.4). Aprovechando esta brecha, se inyectó una secuencia de activación en el nombre de usuario durante la autenticación FTP, lo que abrió de forma inmediata una consola con privilegios de superusuario en el puerto 6200, permitiendo el acceso total a la máquina como root en un solo paso.

## Despliegue
Después de descomprimir el zip de la maquina la desplegaremos usando "sudo bash autodeploy.sh FirstHacking.tar" 

<img width="823" height="211" alt="01_Despliegue" src="https://github.com/user-attachments/assets/f3e38b46-d969-49f1-9cd6-fa30a439f721" />



## 1. Reconocimiento
Se utiliza la herramienta NMAP para descubrir los puertos abiertos en el sistema y ver sus versiones

usaremos el siguiente comando: nmap -vvv -T4 -sV -Pn --open -p- 172.17.0.2

<img width="1182" height="736" alt="02_nmap" src="https://github.com/user-attachments/assets/37f178fd-46b6-4051-b5fe-4384b315ed5d" />


Podemos ver que el único puerto abierto es el FTP (21), investigamos la versión del mismo y descubrimos que tiene una vulnerabilidad conocida.
  
```bash
-vvv = Muestra los resultados en pantalla en tiempo real a medida que los descubre

-T4 = Acelera la velocidad del escaneo, este al ser un entorno controlado lo permitimos

-sV = Detecta la versión exacta de los servicios que se ejecutan en los puertos abiertos

-Pn = Omite la fase de descubrimiento de host mediante ping (ICMP), asumiendo que la máquina objetivo está activa

--open = Muestra únicamente los puertos que se encuentran abiertos, filtrando los cerrados o filtrados

-p- = Escanea el rango completo de puertos (del 1 al 65535)
```


## 1.1. Buscamos información de la versión de ftp y explicamos su vulnerabilidad
Buscando información encontramos que el puerto tiene una vulnerabilidad conocida como CVE-2011-2523, siendo este puerto vulnerable a una puerta trasera que permite la ejecución
de comandos remotamente

Tenemos la posibilidad de conectarnos con una puerta trasera ya que si nos conectamos a la ip de la maquina victima con el servicio ftp, si a la hora de introducir el usuario,
en el mismo nombre del usuario ponemos: :) , se abrirá el puerto 6200 de la maquina victima con permisos de root


## 2. Explotamos la vulnerabilidad
Ya conociendo la vulnerabilidad vamos a explotarla

<img width="954" height="188" alt="03_ftp" src="https://github.com/user-attachments/assets/df54a5f4-1951-4a1c-8a71-588821255b4e" />

Después de introducir un usuario con :) y una contraseña aleatoria se abrirá el puerto en la maquina victima

## 3. Conectarnos a la maquina victima
Con netcat nos conectamos a la maquina victima atraves del puerto 6200

<img width="658" height="105" alt="04_conectarnos-nc" src="https://github.com/user-attachments/assets/6a2c65ad-ccd9-4589-89ba-66b91069dea2" />


## 4. Root

Ya conectados ejecutaremos un whoami para verificar que somos root en la maquina victima

<img width="641" height="128" alt="05_rooteada" src="https://github.com/user-attachments/assets/63c91a6b-919a-4d3b-82f2-8107293fee3f" />

Podemos observar que somos root

## 5. Conclusión

En primer lugar, el vector de entrada demuestra el grave peligro de mantener software obsoleto o con vulnerabilidades críticas en los sistemas. La presencia de la famosa puerta trasera en el servicio FTP (CVE-2011-2523) permitió saltarse por completo los mecanismos tradicionales de autenticación y enumeración de usuarios simplemente enviando un carácter específico :) en el campo de inicio de sesión.

Por último, esta vulnerabilidad resalta la importancia de actualizar constantemente los servicios del sistema operativo y deshabilitar o desinstalar demonios antiguos que ya no sean estrictamente necesarios, ya que un solo servicio mal configurado o vulnerable puede comprometer toda la seguridad de la máquina de forma directa e inmediata.

La máquina ha sido vulnerada con éxito.
