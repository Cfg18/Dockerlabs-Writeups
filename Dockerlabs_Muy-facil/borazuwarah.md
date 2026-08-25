# Writeup: BorazuwarahCTF

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
Después de descomprimir el zip de la maquina la desplegaremos usando "sudo bash autodeploy.sh borazuwaractf.tar" 

<img width="993" height="492" alt="01_Despliegue" src="https://github.com/user-attachments/assets/e4c125df-c6eb-41f5-9c15-262e244b1100" />



## 1. Reconocimiento
Se utiliza la herramienta NMAP para descubrir los puertos abiertos en el sistema y ver sus versiones

usaremos el siguiente comando: nmap -vvv -T4 -sV -Pn --open -p- 172.17.0.2

<img width="1401" height="757" alt="02_escaneo_nmap" src="https://github.com/user-attachments/assets/fdf60ee9-270d-4943-b52f-d48d61e46b87" />

  
```bash
-vvv = Muestra los resultados en pantalla en tiempo real a medida que los descubre

-T4 = Acelera la velocidad del escaneo, este al ser un entorno controlado lo permitimos

-sV = Detecta la versión exacta de los servicios que se ejecutan en los puertos abiertos

-Pn = Omite la fase de descubrimiento de host mediante ping (ICMP), asumiendo que la máquina objetivo está activa

--open = Muestra únicamente los puertos que se encuentran abiertos, filtrando los cerrados o filtrados

-p- = Escanea el rango completo de puertos (del 1 al 65535)
```

## 2. Inspección del servicio HTTP
Revisando el servicio http lo primero que encontraremos será una imagen de un huevo kínder

<img width="763" height="859" alt="03_Imagen_pagina" src="https://github.com/user-attachments/assets/5be6c9df-d124-40a5-8edd-117b99b7d44a" />

## 3. Estenografía
Usando la herramienta exiftool inspeccionamos los metadatos de la imagen, encontrando información sensible, un nombre de usuario

<img width="862" height="523" alt="06_exiftool" src="https://github.com/user-attachments/assets/23182cd6-7678-47a7-822d-f09e3d85a94d" />

## 4. Fuerza bruta
Usamos la herramienta hydra atraves del puerto 22 (ssh) con el nombre de usuario borazuwarah

<img width="950" height="378" alt="07_Hydra" src="https://github.com/user-attachments/assets/5303cc06-9685-4cee-9cdc-c6daf5bd349c" />

```bash
-l = Asignamos un nombre de usuario
-P = Elegimos el archivo para probar las contraseñas, usaremos rockyou.txt
```
Encontramos una coincidencia siendo 123456 de contraseña para borazuwarah

## 5. Nos conectamos a la maquina victima

Ya con el usuario y la contraseña nos conectamos por ssh

<img width="942" height="343" alt="08_ssh" src="https://github.com/user-attachments/assets/ffba3395-bece-4c8c-9823-bf0e722323be" />

Conseguimos conectarnos satisfactoriamente a la maquina victima

## 6. Shell remota

Ya dentro de la maquina ejecutamos "sudo -l" para saber si tenemos la posibilidad de ejecutar algo con permisos de superusuario

<img width="946" height="188" alt="09_sudo-L" src="https://github.com/user-attachments/assets/55ba371f-0a0a-4250-b96c-c7b21b8c4db3" />

Podemos ver que tenemos la posibilidad de ejecutar bash con permisos de usuario sin contraseña

## 7. Root

Ejecutamos "sudo /bin/bash"

<img width="956" height="89" alt="10_root" src="https://github.com/user-attachments/assets/71324840-a1ee-4119-b926-dc76a2d00aa2" />

Después de ejecutar este comando podemos ver que ya hemos ganado acceso a root

## 8. Conclusion

En primer lugar, el vector de entrada demuestra el peligro de la fuga de información a través de recursos públicos. Mantener datos sensibles guardados en los metadatos de las imágenes alojadas en la web permitió obtener un nombre de usuario válido de manera directa, omitiendo la necesidad de realizar una enumeración web agresiva. A esto se le suma el uso de una contraseña extremadamente débil en el servicio SSH, lo que dejó la puerta abierta a un ataque de fuerza bruta exitoso en cuestión de segundos utilizando un diccionario estándar.

Por último, el factor crítico que facilitó la toma del control total de la máquina fue una grave mala configuración en la directiva de sudoers. Otorgar permisos a un usuario sin privilegios para ejecutar una shell interactiva como superusuario , permitiendo una escalada de privilegios a root inmediata.

La maquina ha sido vulnerada con exito


