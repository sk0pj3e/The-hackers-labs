nos entregan 3 archivos pcap y un pdf con instrucciones 

la instrucciones dicen:

      -¡Tu primer desafío como analista de ciberseguridad! Es tu primer día como analista junior en el departamento de ciberseguridad de La Corporación, una organización tecnológica que maneja datos críticos y confidenciales para clientes en todo el mundo. La Corporación ha sido objeto de constantes intentos de intrusión en las últimas semanas, y el equipo de seguridad está bajo presión para evitar cualquier brecha. Tu jefe, un veterano de la ciberseguridad conocido por su actitud directa y su poca paciencia para los novatos, te ha dado tu primera tarea. Mientras te entrega tres registros de tráfico de red capturados durante el fin de semana, te dice con una sonrisa irónica: "Estos logs podrían contener algo interesante… o no. Es tu trabajo averiguarlo. Si logras identificar algún incidente o actividad sospechosa, tal vez merezcas estar aquí. Si no encuentras nada… bueno, siempre hay trabajo en el departamento de soporte." Con los ojos de tus compañeros observándote y el reloj corriendo, sabes que esta es tu oportunidad para demostrar que tienes lo necesario para formar parte de este equipo. Objetivo Tu tarea como nuevo analista es: 1. Analizar los tres registros(viernes.pcap, sabado.pcap, domingo.pcap) 2. Responder a preguntas clave basadas en tus hallazgos.

	
> En el archivo de domingo.pcap nos muestra esta linea
	
	--> 0.086308	192.168.1.100	192.168.10.81	HTTP	100	GET /CVE-2021-41773.sh HTTP/1.1 

![image](https://github.com/user-attachments/assets/a5be5bc1-1574-4956-bf1b-a77ace65187b)


buscando al vulnerabilidad que intentan explotar es esta:
> https://www.exploit-db.com/exploits/50383  
 
Apache HTTP Server 2.4.49 - Path Traversal & Remote Code Execution (RCE)

![image](https://github.com/user-attachments/assets/d1e07b90-5e89-45d2-8486-e536e646b467)


como se que intentan ingresar por el ssh filtrare en wireshark por ese puerto:
> ip.src == 192.168.1.100 && tcp.port == 22

![image](https://github.com/user-attachments/assets/4a9871bf-8545-40bb-8247-00b191a68cd1)


nos muestra que son 10 intentos de ingreso por el puerto 22 o sea el SSH

para ver que tipo de ataque se intento se utiliza:
> http.request.method == GET && ip.src == 192.168.1.100

nos muestra:

![image](https://github.com/user-attachments/assets/b8f04829-c4d1-48f0-b91a-57fe0f4d5a40)

lo importante es: Host: attacker.internal\r\n 

y lo que encontré del ataque es: 

--> Este ataque esta relacionado con la inyección de encabezados HTTP, en este caso el atacante manipulo el encabezado "host" de una solicitud HTTP para redirigir a una dirección maliciosa o sea attacker.internal y esto puede ser usado para tres importantes ataques:

1.- Ataque de server-Side request forgery (SSRF): Engañar al servidor para que realice solicitudes a sistemas internos

2.- Cache Poisoning: Manipular el almacenamiento en cache para servir respuestas maliciosas a futuros usuarios

3.- Cross-Site Scripting (XSS): Si el encabezado manipulado se refleja en respuestas sin sanitizar.  

> Funcionamiento de los ataques:

SSRF: 
1.- El atacante envía una solicitud maliciosa al servidor, manipulando un parámetro que define una URL. 

por ejemplo con burpsuite se puede modificar: 
> GET /fetch?url=http://127.0.0.1/admin

lo redirigió hacia una URL diferente, lo que hace es que el servidor hace la solicitud y devuelve el contenido por ejemplo "/etc/passwd" y eso se redirige al atacante.

solución: 

1.- Validar y sanitizar las URLs ingresadas.  
2.- Restringir las solicitudes del servidor a dominios confiables 
3.- Implementar listas blancas para URLs.

