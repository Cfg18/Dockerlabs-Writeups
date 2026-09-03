# Writeup: VacacionesCTF

* **Autor:** Cfg18
* **Plataforma:** DockerLabs
* **Dificultad:** Muy Fácil
* **SO:** Linux
* **Categoría:** Web / Linux
* **Fecha:** 25/08/2026

## Resumen Ejecutivo (TL;DR)
Se realizó la auditoría de seguridad sobre la máquina objetivo identificando dos puertos abiertos: HTTP (80) y SSH (22).
Mediante el análisis de metadatos EXIF con exiftool en una imagen alojada en la web, se descubrió el usuario del sistema (borazuwarah).
Posteriormente, se ejecutó un ataque de fuerza bruta con Hydra sobre el servicio SSH para obtener la contraseña de acceso.
Una vez dentro del sistema, se auditó la configuración de privilegios, detectando que el usuario podía ejecutar
/bin/bash con permisos de superusuario sin contraseña, permitiendo la escalada directa a root

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
-dir = Seleccionamos el modo enumeración de directorios y archivos

-u = Especifica la url de la pagina que vamos a escanear

-w = Definimos el diccionario que vamos a usar

```

<img width="888" height="974" alt="05_Forbidden" src="https://github.com/user-attachments/assets/05af664a-6efa-4606-ab08-b5d186a297ba" />

Encontramos que no tenemos los permisos suficientes como para estar en la pagina

## 4. Curl
Usamos Curl para obtener mas informacion del servicio web y encontramos un mensaje entre desarrolladores

<img width="950" height="248" alt="06_Curl" src="https://github.com/user-attachments/assets/9363559f-c548-4b47-89a4-dc0e0da4245c" />


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
sudo ruby -e ‘exec"/bin/bash"’
```
<img width="872" height="95" alt="11_rubyvuln" src="https://github.com/user-attachments/assets/c49c79cb-ea15-41e3-a0f7-fd0370a077c3" />

Despues de esto, ya tendremos permisos de superusuario

## 9. Conclusion

En primer lugar, el vector de entrada demuestra el peligro de la fuga de información a través de recursos públicos. Mantener datos sensibles guardados en los metadatos de las imágenes alojadas en la web permitió obtener un nombre de usuario válido de manera directa, omitiendo la necesidad de realizar una enumeración web agresiva. A esto se le suma el uso de una contraseña extremadamente débil en el servicio SSH, lo que dejó la puerta abierta a un ataque de fuerza bruta exitoso en cuestión de segundos utilizando un diccionario estándar.

Por último, el factor crítico que facilitó la toma del control total de la máquina fue una grave mala configuración en la directiva de sudoers. Otorgar permisos a un usuario sin privilegios para ejecutar una shell interactiva como superusuario , permitiendo una escalada de privilegios a root inmediata.

La maquina ha sido vulnerada con exito


