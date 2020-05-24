![title](https://miro.medium.com/max/644/1*TLyMXNil-OxKMLq8WZt96w.png)

Puedes encontrar el room en  : https://tryhackme.com/room/skynet

## Reconocimiento

Primero se identificaron los puertos abiertos con NMAP

![skynet](https://miro.medium.com/max/707/1*qKFE46r9__4Nq6H8u_pfzA.png)

Una vez identificados los puertos, se identificaron las versiones de los servicios activos en cada uno de los puertos

![skynet](https://miro.medium.com/max/770/1*K8ux8rXkN2_6AeF_EwFHcg.png)

## HTTP

Ejecutando dirb, se identificaron los siguientes directorios

	/index.html 
	/admin 
	/ai 
	/config 
	/css 
	/js 
	/squirrelmail 
	/.htaccess 
	/.hta 
	/.htpasswd 
	/server-status

## SAMBA

Se procedió a enumerar los recursos y usuarios con nmap y smb-enum-*

	Host script results:
	| smb-enum-domains: 
	|   SKYNET
	|     Groups: n/a
	|     Users: milesdyson
	|     Creation time: unknown
	|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
	|     Account lockout disabled
	|   Builtin
	|     Groups: n/a
	|     Users: n/a
	|     Creation time: unknown
	|     Passwords: min length: 5; min age: n/a days; max age: n/a days; history: n/a passwords
	|_    Account lockout disabled
	| smb-enum-sessions: 
	|_  <nobody>
	| smb-enum-shares: 
	|   account_used: guest
	|   \\10.10.51.155\IPC$: 
	|     Type: STYPE_IPC_HIDDEN
	|     Comment: IPC Service (skynet server (Samba, Ubuntu))
	|     Users: 3
	|     Max Users: <unlimited>
	|     Path: C:\tmp
	|     Anonymous access: READ/WRITE
	|     Current user access: READ/WRITE
	|   \\10.10.51.155\anonymous: 
	|     Type: STYPE_DISKTREE
	|     Comment: Skynet Anonymous Share
	|     Users: 0
	|     Max Users: <unlimited>
	|     Path: C:\srv\samba
	|     Anonymous access: READ/WRITE
	|     Current user access: READ/WRITE
	|   \\10.10.51.155\milesdyson: 
	|     Type: STYPE_DISKTREE
	|     Comment: Miles Dyson Personal Share
	|     Users: 0
	|     Max Users: <unlimited>
	|     Path: C:\home\milesdyson\share
	|     Anonymous access: <none>
	|     Current user access: <none>
	|   \\10.10.51.155\print$: 
	|     Type: STYPE_DISKTREE
	|     Comment: Printer Drivers
	|     Users: 0
	|     Max Users: <unlimited>
	|     Path: C:\var\lib\samba\printers
	|     Anonymous access: <none>
	|_    Current user access: <none>
	| smb-enum-users: 
	|   SKYNET\milesdyson (RID: 1000)
	|     Full name:   
	|     Description: 
	|_    Flags:       Normal user account

## Acceso Inicial

El directorio Anonymous permitía conexiones anónimas a través del smbclient se accedió al directorio y se procedió a listar los recursos

![skynet](https://miro.medium.com/max/1400/1*RFQUWGUNvd35jbpwH-NcKw.png)

Dentro de la sesión de smbclient smb: \> get attention.txt para descargar el archivo de forma local.

![skynet](https://miro.medium.com/max/1400/1*-oRJpu0A-yxc_M4NFhUHVw.png)

En el directorio logs, se encontró 3 archivos log1.txt,log2.txt y log3.txt, el primer archivo contenía información necesaria para realizar pruebas.

![skynet](https://miro.medium.com/max/1400/1*cJGxB2fD9zw0IpT_bEluZg.png)

Utilizando el usuario y el listado, se obtuvo acceso a squirrelmail

![skynet](https://miro.medium.com/max/1400/1*iX8AVHAZUJeG5AEYJYFbMA.png)

Revisando los correos se obtuvo información para iniciar sesión

![skynet](https://miro.medium.com/max/1400/1*649ewisn54donnIt_sMHmA.png)

con smbclient se accedió al dicterio de milesdyson

![skynet](https://miro.medium.com/max/1400/1*pDn9GtQ59jLF8nVu4q1fFQ.png)

Solo existía un directorio /notes

![skynet](https://miro.medium.com/max/1400/1*LCsiL0ZUw_Gg9aoTHt2AUw.png)

Se descargó el archivo important.txt

![skynet](https://miro.medium.com/max/1400/1*Xg80CiaUrzeNotaJodnFTg.png)

Accediendo al archivo important.txt , se identificó un nuevo directorio

![skynet](https://miro.medium.com/max/1076/1*z9BDjcTsInTjvyjyz4euug.png)


![skynet](https://miro.medium.com/max/770/1*UvCNBcrnz7Z_s1USmlg4KQ.png)

Mediante dirb se identificó el directorio /administrator

![skynet](https://miro.medium.com/max/770/1*2qPr1P8DGxQe7krFRdjvrQ.png)

En exploit-db se identificó una vulnerabilidad para acceder a información sin estar autenticado.

![skynet](https://miro.medium.com/max/727/1*Wbf6lvvfRuAJIU_Nv2ZYrw.png)

Mediante una reverse-shell.php se obtuvo acceso inicial

![skynet](https://miro.medium.com/max/770/1*1Cnc6gEydAJx3YSJGMlGCw.png)

![skynet](https://miro.medium.com/max/770/1*vLuXpjz5IV3QYaQ2AtRMDg.png)

## Escalada de privilegio

Mediante la herramienta linPEAS se procedió a enumerar recursos, servicios, SUID, tareas programadas para determinar el método de escalada de privilegio.

La herramienta utilizada fue:

https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS

Se identificó una tarea que se ejecuta con el usuario root, esta tarea se ejecutaba a través del script backup.sh

![skynet](https://miro.medium.com/max/685/1*v_9Wk8CaBXcAoxQR7kO5uQ.png)

Al acceder al directorio de milesdyson y listar el archivo backup.sh se observo el uso de comodín con el binario TAR ;)

![skynet](https://miro.medium.com/max/750/1*7imqUTBY3Qkw-I8E42wh6Q.png)

La técnica utilizada para la escalada fue Exploiting Wildcard y SUID la primera para otorgar S a un binario en especifico a Find. Exploiting wildcard afecta a algunos binarios en particular : chown, tar, rsync al utilizar nombres específicos permite la ejecución de código arbitrario.

El detalle de la técnica se encuentra en :

https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/

Es importante que la tecnica sea aplicada en el directorio al cual hace referencia el script backup.sh

#### Método utilizado:
1. Se Creó el archivo test.sh que ejecuta el comando “chmod u+s” al binario find para activar el SUID
2. Se creó el archivo “ — checkpoint-action=exec=sh test.sh” (ejecuta una accion, tambien se puede reemplazar “sh test.sh” por /bin/sh para obtener una shell)
3. Se creó el archivo “ — checkpoint=1”

![skynet](https://miro.medium.com/max/770/1*IuJAynErZWvbDme5XTg44w.png)
	
Detalle de todos los parámetros de TAR:

http://man.he.net/?topic=tar&section=all

Se puede observar los archivos creados

![skynet](https://miro.medium.com/max/770/1*CN4IVjbp0ukp9Dn_eyv7mA.png)

Se debe esperar que se ejecute la tarea programada

![skynet](https://miro.medium.com/max/603/1*q9fPTH8UTYJWIegJlX-LcQ.png)

Verificamos si el binario find cuenta con el Suid activo y con este binario se escalo privilegios

![skynet](https://miro.medium.com/max/354/1*DxsHN0iSMxJ1HTr1HHHgVA.png)



