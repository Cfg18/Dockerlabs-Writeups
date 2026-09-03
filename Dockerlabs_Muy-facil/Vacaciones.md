# Writeup: VacacionesCTF

* **Autor:** Cfg18
* **Plataforma:** DockerLabs
* **Dificultad:** Muy Fácil
* **SO:** Linux
* **Categoría:** Web / Linux
* **Fecha:** 25/08/2026

## Resumen Ejecutivo (TL;DR)
Se realizó la auditoría de seguridad sobre la máquina objetivo identificando los puertos abiertos correspondientes a HTTP (80) y SSH (22). Tras realizar un análisis de enumeración web con gobuster y obtener información relevante mediante curl, se descubrió un mensaje oculto/comentario con pistas entre desarrolladores. Posteriormente, se ejecutó un ataque de fuerza bruta con hydra sobre el servicio SSH utilizando las pistas obtenidas para dar con el usuario y la contraseña de acceso inicial (camilo).

Una vez dentro del sistema como usuario estándar, se inspeccionaron los buzones de correo locales en la ruta /var/mail, donde se encontraron credenciales adicionales correspondientes al usuario juan. Tras acceder con este nuevo usuario y auditar los permisos de sudo , se detectó que juan podía ejecutar el intérprete de comandos ruby con privilegios de superusuario sin requerir contraseña. Finalmente, mediante un comando de ejecución de procesos en Ruby, se logró la escalada de privilegios directa a root.

## Despliegue
Después de descomprimir el zip de la maquina la desplegaremos usando "sudo bash autodeploy.sh Vacaciones.tar" 

<img width="871" height="174" alt="01_Despliegue" src="https://github.com/user-attachments/assets/b194c244-28c6-4adb-82e2-d45228a6dcf0" />



## 1. Reconocimiento
Se utiliza la herramienta NMAP para descubrir los puertos abiertos en el sistema y ver sus versiones

usaremos el siguiente comando: nmap -vvv -T4 -sV -Pn --open -p- 172.17.0.2

<img width="1566" height="762" alt="02_nmap" src="https://github.com/user-attachments/assets/b5da4776-1942-4ac3-9154-e4e5ecc7dad2" />


  
```bash
-vvv = Muestra los resultados en pantalla en tiempo real a medida que los descubre

-T4 = Acelera la velocidad del escaneo, este al ser un entorno controlado lo permitimos

-sV = Detecta la versión exacta de los servicios que se ejecutan en los puertos abiertos

-Pn = Omite la fase de descubrimiento de host mediante ping (ICMP), asumiendo que la máquina objetivo está activa

--open = Muestra únicamente los puertos que se encuentran abiertos, filtrando los cerrados o filtrados

-p- = Escanea el rango completo de puertos (del 1 al 65535)
```

## 2. Inspección del servicio HTTP
Revisando el servicio http descubriremos una pagina en blanco

<img width="907" height="1008" alt="03_Puerto80" src="https://github.com/user-attachments/assets/0a7c2fb9-9741-4a73-a46b-56c0f476c1ac" />


## 3. enumeración de directorios y archivos web
Usamos gobuster para enumerar el sitio web

<img width="1316" height="512" alt="04_gobuster" src="https://github.com/user-attachments/assets/782855ba-74fb-4bf2-96d4-f8409e0465f4" />

```bash
dir = Seleccionamos el modo enumeración de directorios y archivos

-u = Especifica la url de la pagina que vamos a escanear

-w = Definimos el diccionario que vamos a usar

```

<img width="888" height="974" alt="05_Forbidden" src="https://github.com/user-attachments/assets/05af664a-6efa-4606-ab08-b5d186a297ba" />

Encontramos que no tenemos los permisos suficientes como para estar en la pagina

## 4. Curl
Usamos Curl para obtener mas informacion del servicio web y encontramos un mensaje entre desarrolladores

<img width="950" height="248" alt="06_Curl" src="https://github.com/user-attachments/assets/9363559f-c548-4b47-89a4-dc0e0da4245c" />

```bash
-i = Incluye tambien las  cabeceras y el body de la pagina
```

## 5. Fuerza bruta

Usado el curl y descubierto que Camilo sea posiblemente un usuario usaremos hydra y probaremos creedenciales, probamos con "Camilo" 
como nombre de usuario pero no da resultados así que probamos luego con "camilo" como nombre de usuario resultando en una coincidencia

<img width="874" height="821" alt="07_Hydra" src="https://github.com/user-attachments/assets/b174d714-1650-4d03-b49a-ee067eef8e5b" />


## 6. Shell remota

Ya dentro de la maquina proamos a ejecutar sudo -l resultando que el usuario no lo puede ejecutar, recordando lo que habíamos visto antes
en el curl, sabemos que a Camilo le habian dejado un correo importante asi que buscamos en carpetas del sistema

<img width="873" height="339" alt="09_var" src="https://github.com/user-attachments/assets/0c1b6a50-effe-4c43-b4c4-d2fb180d4a4a" />

Vemos que Juan nos ha dejado un mensaje con sus creedenciales

## 7. Escalado de privilegios

Entramos como el usuario "juan" y ejecutamos "sudo -l" , mostrandonos que el usuario puede ejecutar ruby con permisos de superusuario

<img width="883" height="440" alt="10_sshjuan" src="https://github.com/user-attachments/assets/c3d658df-8d79-4152-9e8c-402655500aa1" />

## 8.Root

Como ya sabemos que el usuario juan puede ejecutar ruby, solamente tendremos que ejecutar

```bash
sudo ruby -e 'exec"/bin/bash"'
```
<img width="872" height="95" alt="11_rubyvuln" src="https://github.com/user-attachments/assets/c49c79cb-ea15-41e3-a0f7-fd0370a077c3" />

Despues de esto, ya tendremos permisos de superusuario

## 9. Conclusion

En primer lugar, el vector de entrada demuestra el peligro de exponer información y pistas en directorios o respuestas HTTP mal protegidas o comentadas en el código web. Combinar esto con la reutilización de credenciales débiles o fácilmente adivinables facilitó que un atacante pudiera superar el servicio SSH mediante un ataque de fuerza bruta estándar.

Asimismo, la vulnerabilidad derivada del almacenamiento de información sensible en los buzones locales del sistema (/var/mail), permitió un movimiento lateral limpio hacia otro usuario del sistema.

Por último, el factor crítico que propició la toma de control total de la máquina fue una mala configuración severa en los permisos del archivo sudoers. Permitir que un usuario sin privilegios ejecute un lenguaje de scripting potente como Ruby con permisos de superusuario sin restricciones ni contraseñas habilitó una vía directa y rápida de escalada de privilegios a root.

La máquina ha sido vulnerada con éxito.
