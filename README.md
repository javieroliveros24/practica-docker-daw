\# Practica DAW Nginx y Apache con Docker

Diagrama de arquitectura hecho con la herramienta [Draw.io](http://Draw.io)  
<img width="604" height="237" alt="62DCFD22-A7C4-48DA-86DD-7A2810DDEDEA_4_5005_c" src="https://github.com/user-attachments/assets/daec4f44-412a-4b04-bc78-2940c0bed104" />


En esta práctica he montado una arquitectura de dos capas usando Docker Compose para separar la parte pública de la privada.

La idea principal es tener un proxy Nginx al frente que recibe el tráfico y por detrás un servidor Apache aislado en una red interna que no se puede tocar desde fuera.

\#\# Arquitectura del sistema

El proyecto se divide en dos contenedores principales:

\- nginx\_edge que es el proxy inverso. Está expuesto en el puerto 80 del ordenador y se encarga de recibir las peticiones de fuera, meter cabeceras de seguridad y pasarselas a Apache.  
\- apache\_backend que es el servidor web que tiene la aplicación. Está metido en una red bridge interna llamada backend-net que no tiene puertos abiertos hacia la máquina. De esta forma nadie puede saltarse el proxy.

\!\[Arquitectura del Sistema\](arquitectura.png)

\#\# Seguridad y logs

Para la parte de seguridad en Nginx he quitado la versión del servidor para que no se sepa que software hay exactamente y he metido cabeceras típicas contra ataques de clickjacking o manipulación de contenido.

En Apache la parte privada /privado/ está protegida. Si intentas entrar normal salta un error 403\. Para que te deje pasar hay que mandarle la cabecera X-Auth-Token con el valor DAW2026 y entonces devuelve un 200\.

También he solucionado el problema de la IP real en los registros. Como Apache está detras de Nginx de normal solo guardaría la IP interna del proxy. He usado el módulo mod\_remoteip junto con X-Forwarded-For para que en el access.log quede guardada la IP real del cliente que hace la consulta para cumplir con la auditoría.

\#\# Cómo desplegarlo

Para arrancar todo el entorno solo hay que ejecutar este comando en la terminal desde la carpeta del proyecto:

\-docker compose up \-d

Para comprobar que los dos contenedores estan funcionando correctamente:

\-docker compose ps

Y si se quiere apagar todo:

docker compose down

\#\# Pruebas que he realizado

Para probar la zona privada sin token y comprobar que da error 403:

curl \-I http\://localhost/privado/

Para probar meter el token correcto y ver que entra con un 200 OK:

curl \-H "X-Auth-Token: DAW2026" \-I http\://localhost/privado/

Para revisar los logs en Apache y confirmar que esta pillando la IP real del cliente:

docker compose logs apache-backend

\#\# Estructura de archivos

.gitignore  
README.md  
arquitectura.png  
default.conf  
docker-compose.yml  
apache/  
  logs/  
    .gitkeep

Captura de **Levantar los contenedores de Docker**
<img width="601" height="449" alt="6FFCB365-1DC5-48AC-B7AF-09D93FFE390C" src="https://github.com/user-attachments/assets/4dc478de-92a7-4507-a342-f4819419e2fa" />



Captura de **Petición a Nginx Directo**:  
<img width="600" height="172" alt="E966B88A-3104-4131-8976-7A768FDE145E_4_5005_c" src="https://github.com/user-attachments/assets/7477403d-ae49-4d1c-8bbd-46a6f36aebc1" />


**Captura de Petición a Apache a través del Proxy**:  
<img width="598" height="189" alt="2BE9AB15-E22B-49DD-80F7-8FEC5D06DCF0_4_5005_c" src="https://github.com/user-attachments/assets/d91a021d-ce74-4be7-989a-3dfa80f14e03" />


**Captura de Acceso sin la cabecera personalizada (debe dar error 403\)**  
<img width="594" height="138" alt="59E3DAF5-7B9C-4C9B-BBAF-91C5D20812B7_4_5005_c" src="https://github.com/user-attachments/assets/13fcf5ba-ef36-49ef-805e-e3b40f9a858c" />


**Captura de Acceso con la cabecera X-Auth-Token: DAW2026**  
<img width="600" height="198" alt="B9232530-2BE9-48A9-8B39-661673028FAB_4_5005_c" src="https://github.com/user-attachments/assets/54611b10-0b0d-4e8c-892e-c890f6a3b5ed" />


**Captura de los logs en la terminal**  
<img width="604" height="228" alt="EAC5017C-E29F-437E-81FD-FC3BABA766E2_4_5005_c" src="https://github.com/user-attachments/assets/b01b4913-3505-47d7-9357-78928332e11e" />


