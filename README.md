# Evaluación de Habilidades - Enumeración y Ataques de Active Directory

Este repositorio contiene la documentación de las evaluaciones de habilidades del módulo "Enumeración y Ataques de Active Directory" del curso de Hack The Box. Incluye la resolución detallada de escenarios prácticos, el uso de diversas herramientas y la aplicación de técnicas para comprometer dominios en entornos simulados de Active Directory.

## Herramientas Utilizadas

Durante el desarrollo de estas evaluaciones, se emplearon las siguientes herramientas:

- **Responder**: Para envenenar la red y capturar hashes NTLM.
- **Hashcat**: Para descifrar hashes y obtener contraseñas en texto claro.
- **Evil-WinRM**: Para acceso remoto a sistemas Windows con privilegios.
- **Rpcclient**: Para enumerar usuarios de dominio.
- **Kerbrute**: Para ataques de password spraying.
- **CrackMapExec**: Para exploración y explotación de recursos compartidos.
- **Smbmap**: Para enumerar y descargar archivos desde recursos compartidos.
- **Mssqlclient.py**: Para interactuar con bases de datos MSSQL.
- **GodPotato**: Para aprovechar el privilegio SeImpersonatePrivilege y escalar privilegios.
- **Mimikatz**: Para obtener credenciales en texto claro, hashes y realizar ataques DCSync.
- **BloodHound**: Para analizar relaciones y permisos en Active Directory.
- **Psexec.py**: Para ejecutar comandos de manera remota utilizando Pass-The-Hash.

Cada herramienta fue seleccionada y utilizada estratégicamente para cumplir con los objetivos de cada escenario.
