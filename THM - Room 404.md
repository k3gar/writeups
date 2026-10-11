[THM - Room 404](https://tryhackme.com/room/hh-room404-804573bf)

Este desafío trata sobre los riesgos de seguridad relacionados a la exposición de metadatos de control de acceso en servidores web públicos.

Al realizar un análizis con [[Nmap]] se identificó vulnerabilidades básicas relacionadas a la exposición de un repositorio git.

> nmap -sV -sC -oN nmap/initial * Target *

![[nmapResult.png]]

Al mismo tiempo, validamos el contenido web disponible con [[Feroxbuster]] 

![[feroxbuster.png]]

Con Feroxbuster necesitamos un directorio. En este caso usamos el de seclists, el cual podemos descargar con el comando `sudo apt -y install seclists` 

Al validar el repositorio expuesto, utilizamos [[git-dumper]] para resonstruir de forma local el repositorio con el comando `git-dumper http://10.64.187.19:8080/.git loot` 

loot es el nombre del fichero en que queremos guardar el registro.

![[Pasted image 20260913222633.png]]

Al tener reconstruido el repositorio, podemos revisar su contenido, entre ellos, el archivo README.md

![[Pasted image 20260913222730.png]]