# Writeup: BorazuwarahCTF

* **Autor:** d1egu1nh10
* **Plataforma:** DockerLabs
* **Dificultad:** Muy Fácil
* **SO:** Linux
* **Categoría:** Web / Linux
* **Fecha:** 25/08/2026

## Resumen Ejecutivo (TL;DR)
Se realizó la auditoría de seguridad sobre la máquina objetivo identificando dos puertos abiertos: HTTP (80) y SSH (22).
Mediante el análisis de metadatos EXIF con exiftool en una imagen alojada en la web, se descubrió el usuario del sistema (borazuwarah).
Posteriormente, se ejecutó un ataque de fuerza bruta con Hydra sobre el servicio SSH para obtener la contraseña de acceso.
Una vez dentro del sistema, se auditó la configuración de privilegios mediante sudo -L, detectando que el usuario podía ejecutar
/bin/bash con permisos de superusuario sin contraseña, permitiendo la escalada directa a root

## 1. Reconocimiento
Se utiliza la herramienta NMAP para descubrir los puertos abiertos en el sistema y ver sus versiones

	nmap -vvv -T4 -sV -Pn --open -p- 172.17.0.2
  
```bash
-vvv = Muestra los resultados en pantalla en tiempo real a medida que los descubre

-T4 = Acelera la velocidad del escaneo, este al ser un entorno controlado lo permitimos

-sV = Detecta la versión exacta de los servicios que se ejecutan en los puertos abiertos

-Pn = Omite la fase de descubrimiento de host mediante ping (ICMP), asumiendo que la máquina objetivo está activa

--open = Muestra únicamente los puertos que se encuentran abiertos, filtrando los cerrados o filtrados

-p- = Escanea el rango completo de puertos (del 1 al 65535)
```

## 2. Inspección del servicio HTTP
Revisando el servicio http lo primero que encontraremos será una imagen de un huevo kinder

![Imagen web](03_imagen_pagina.png)
