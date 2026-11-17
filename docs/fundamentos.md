# HomeLab 01 — Fundamentos y glosario

Adaptación para lectura en GitHub del [documento original](03_HomeLab01_Fundamentos.docx). Se conservan sus contenidos, tablas y figuras; la presentación y la paginación son distintas.

[Volver al proyecto](../README.md)

HOMELAB 01

Fundamentos y glosario aplicado

Conceptos para comprender el laboratorio y estudiar sin el chat

Máximo Rodríguez

Redes · Linux · Administración · Fundamentos de seguridad

Edición de estudio | 14 de noviembre de 2026

Documento 03 de 3

### Cómo uso este documento

Relaciono cada concepto con una observación del laboratorio. Las ampliaciones de estudio no representan prácticas adicionales realizadas.

### Contenido

1. Qué construí y qué significa aislar una red

2. IPv4, prefijos, rutas y siguiente salto

3. ARP, vecinos y resolución local

4. Encapsulación, ICMP y lectura de capturas

5. TCP: conexión, puertos y cierre

6. SSH: cifrado e identidad en dos direcciones

7. Linux: programa, proceso, servicio y socket

8. Usuarios, permisos y mínimo privilegio

9. Firewall, persistencia y diagnóstico con evidencia

10. Repaso autónomo y próximos estudios

## 1. Qué construí y qué significa aislar una red

Construí dos máquinas virtuales para practicar redes y administración sin necesitar dos computadoras físicas. El hipervisor presenta CPU, memoria, disco y tarjetas de red virtuales a cada sistema invitado. Dentro de la VM, Linux administra esos recursos como dispositivos propios; el host sigue siendo la computadora Windows que los sostiene.

Elegí dos adaptadores por VM para separar propósitos. La red interna hlab01 transportó la comunicación Kali–Ubuntu. NAT permitió salida externa desde cada VM, por ejemplo para paquetes del sistema o el ping a 8.8.8.8. No configuré a una VM como router de la otra.

| Concepto | Cómo lo relaciono con el laboratorio |
| --- | --- |
| Host | Windows y la computadora física que ejecutan VirtualBox. |
| Guest o invitado | Kali o Ubuntu Server dentro de una VM. |
| NIC virtual | Tarjeta presentada al invitado; la comparo por MAC con el adaptador de VirtualBox. |
| Red interna | Segmento virtual compartido por adaptadores con el mismo nombre de red. |
| NAT | Traducción y tratamiento del tráfico externo por la infraestructura virtual; las IP privadas no se vuelven públicas dentro de la VM. |
| Bridge | Modo diferente que conectaría el invitado a la red externa mediante el host; no lo usé para hlab01. |
| Host-only | Modo distinto que incluye un enlace virtual con el host; no es el modo elegido para hlab01. |
| Dos adaptadores | Dos conexiones de red; no implican reenvío automático entre ellas. |

La red interna no tiene salida a Internet por sí misma, pero las VM sí conservaron otro adaptador con salida NAT. Por eso describo el segmento de prácticas como separado, no a las máquinas completas como desconectadas de Internet. Un nombre de red idéntico y direccionamiento compatible son dos condiciones diferentes que deben cumplirse.

Para estudiar: no identifico el modo de VirtualBox solo por una IP privada. La misma numeración puede aparecer en diferentes modos. Verifico la configuración virtual, la interfaz dentro de Linux y sus rutas.

## 2. IPv4, prefijos, rutas y siguiente salto

Una dirección IPv4 identifica una interfaz en un contexto de red. El prefijo indica qué parte se utiliza para comparar la red. En mi segmento 192.168.50.0/24, los primeros 24 bits corresponden al prefijo y quedan 8 bits para las direcciones dentro de la subred.

| Dato | Valor en hlab01 |
| --- | --- |
| Red | 192.168.50.0/24 |
| Máscara | 255.255.255.0 |
| Kali | 192.168.50.20 |
| Ubuntu | 192.168.50.10 |
| Broadcast de la subred | 192.168.50.255 |
| Rango habitual de hosts en esta /24 | 192.168.50.1 a 192.168.50.254 |

Las direcciones de red y broadcast de esta /24 no las asigno a los hosts. Esa regla se explica para esta subred; no la generalizo sin matices a todos los prefijos y casos especiales de IPv4.

### Decisión de ruta

Linux consulta las rutas para elegir una salida y un siguiente salto. En el caso básico del laboratorio, una coincidencia /24 hacia 192.168.50.0 tiene prioridad sobre la ruta predeterminada /0. Una métrica menor no hace que una ruta menos específica gane automáticamente.

| Destino desde Kali | Interfaz | Siguiente salto IP | MAC necesaria |
| --- | --- | --- | --- |
| 192.168.50.10 | eth1 | El propio Ubuntu, directamente conectado | MAC de enp0s8 de Ubuntu |
| 8.8.8.8 | eth0 | Gateway 10.0.2.2 | MAC del gateway virtual |

La IP destino y la MAC destino responden preguntas diferentes. Para el ping externo, el paquete sigue dirigido a 8.8.8.8 mientras la trama local se entrega al gateway. La MAC del servidor remoto no atraviesa Internet como dirección de destino Ethernet en cada enlace.

La ruta de entrega directa no necesita un gateway intermedio. Dejé el gateway vacío en el perfil interno y evité una ruta predeterminada desde ese perfil para conservar la salida externa por NAT.

### Ampliación conceptual: NAT y routing

Routing decide dónde reenviar un paquete; NAT modifica determinadas direcciones y, según el tipo de traducción, puertos. No son sinónimos. En una salida habitual con NAT, la traducción ocurre fuera del punto donde capturé en eth0 del invitado, por eso allí observé la IPv4 privada de Kali como origen.

Al atravesar un router Ethernet, el paquete se recibe en una trama y se transmite mediante otra en el siguiente enlace. Las MAC corresponden a cada enlace. El paquete IP también puede sufrir cambios, como TTL, y NAT puede modificar direcciones. Por eso no afirmo que todos sus bytes sean idénticos extremo a extremo.

## 3. ARP, vecinos y resolución local

ARP permite obtener una dirección de enlace asociada a una dirección de protocolo, en mi caso la MAC de un vecino IPv4 sobre Ethernet. No convierte los bytes de una IP en una MAC ni consulta un directorio universal: intercambia mensajes en el enlace.

Cuando Kali necesitó alcanzar Ubuntu y no tenía una asociación utilizable, preguntó por 192.168.50.10. La solicitud se envió a ff:ff:ff:ff:ff:ff, el broadcast Ethernet. Ubuntu respondió con su MAC y Kali pudo construir la trama unicast hacia ese vecino.

| Campo | Pregunta que responde |
| --- | --- |
| Ethernet destination | ¿A quién entrego esta trama en el enlace? |
| ARP Sender IP / MAC | ¿Qué direcciones declara el emisor? |
| ARP Target IP | ¿Por qué IPv4 consulto? |
| ARP Target MAC | ¿Qué dirección de enlace tiene el objetivo, si está disponible? En la solicitud inicial puede aparecer en ceros. |
| Opcode | ¿Es una solicitud o una respuesta ARP? |

La MAC broadcast de la cabecera Ethernet y la Target MAC de ARP son campos distintos. Ver ceros en Target MAC no significa que la trama se haya enviado a esa dirección: la cabecera Ethernet contiene el destino de la transmisión.

### Caché de vecinos

Linux conserva asociaciones para evitar una consulta ARP antes de cada paquete. STALE indica que una entrada existente necesita revalidación de alcanzabilidad según su uso; no significa que sea inútil ni que el equipo esté apagado. FAILED indica una resolución o comprobación fallida; su causa requiere contexto.

La ausencia de una consulta ARP justo antes del ping puede explicarse por una entrada utilizable en caché. Para observar nuevamente el intercambio eliminé solo la asociación de Ubuntu y capturé antes de generar el tráfico.

### Qué me enseñó el fallo de .99

Observé solicitudes ARP por 192.168.50.99 sin respuesta. El problema se produjo antes de entregar el Echo Request por Ethernet. Eso es diferente de enviar el ping, recibirlo en el destino y que un firewall lo descarte. Identificar el primer paso que no se completa ordena el diagnóstico.

### Conexión con seguridad

ARP no autentica criptográficamente sus anuncios. Una asociación observada debe interpretarse con el contexto del segmento. El envenenamiento ARP es una posibilidad de abuso relacionada, pero no lo ejecuté en este laboratorio. En IPv6 se utiliza Neighbor Discovery sobre ICMPv6 en lugar de ARP; lo incluyo para no aplicar mecánicamente el procedimiento IPv4 a todo tráfico.

## 4. Encapsulación, ICMP y lectura de capturas

Una trama es una unidad del nivel de enlace. Un paquete IP es una unidad del nivel de red. Un mensaje ICMP puede viajar dentro de ese paquete IP. En mi ping sobre Ethernet, las tres estructuras pertenecen a una transmisión encapsulada.

| Caso | Contenedor exterior | Contenido |
| --- | --- | --- |
| Resolución ARP | Ethernet | Mensaje ARP directamente |
| Ping IPv4 | Ethernet | IPv4, que contiene ICMP |
| Comunicación SSH | Ethernet | IPv4, que contiene TCP y datos del protocolo SSH |

Que ARP lleve direcciones IPv4 entre sus campos no significa que esté encapsulado en un paquete IPv4. La cabecera Ethernet indica el tipo de contenido: en estas capturas 0x0806 identifica ARP y 0x0800 identifica IPv4.

### Solicitud y respuesta

Kali genera un Echo Request y Ubuntu genera un Echo Reply. Son dos mensajes distintos transportados en tramas distintas. En este intercambio local las direcciones de origen y destino se invierten al responder. Identifier y Sequence Number ayudan a relacionar los mensajes; no identifican una única trama que viaja de ida y vuelta.

Si cada mensaje ocupa una trama, cuatro solicitudes y cuatro respuestas suman ocho tramas que transportan ICMP. Las tramas ARP se cuentan aparte. Desplegar Ethernet, IPv4 e ICMP en Wireshark no multiplica ese conteo por tres.

### Qué prueba un ping

Una respuesta ICMP indica que hubo un intercambio de eco entre los extremos observados. No prueba disponibilidad de SSH, autorización de una cuenta ni velocidad máxima del enlace. El tiempo mostrado es ida y vuelta; el TTL es un contador de saltos restante, no un tiempo medido en milisegundos.

Un error de destino inalcanzable debe interpretarse considerando quién lo emite y qué ocurrió antes. En mi prueba .99, la evidencia ARP y la tabla de vecinos explican el fallo local. En otros escenarios el error puede originarse en otro equipo o tener otra causa.

### Captura y filtro

Una PCAPNG conserva lo registrado en un punto de observación. No es una visión absoluta de toda la red. Un filtro de visualización decide qué filas muestro; no necesariamente elimina el resto del archivo. Por eso el PCAP local tiene 15 tramas y el análisis ARP/ICMP considera 12.

El número de trama sirve como referencia dentro de ese archivo. Si exporto solamente algunos paquetes, los índices pueden cambiar. Para una evidencia reproducible guardo nombre de archivo, filtro, número de trama y explicación. Las capturas de pantalla ayudan a leer, pero el PCAP permite volver a inspeccionar los campos.

## 5. TCP: conexión, puertos y cierre

TCP ofrece a las aplicaciones un flujo de bytes ordenado, con control de errores, confirmaciones y retransmisiones. La aplicación no recibe necesariamente un bloque por cada paquete capturado. Una escritura puede dividirse en segmentos y varios segmentos pueden reconstruir un mensaje de aplicación.

Un puerto identifica un extremo de transporte en el contexto de una dirección y un protocolo. En mi sesión, Ubuntu escucha en TCP 22 y Kali utiliza un puerto temporal, como 33648. No confundo ese número con PID, descriptor de archivo o identificador de trama.

| Dato | Ejemplo | Función |
| --- | --- | --- |
| IP cliente | 192.168.50.20 | Dirección de Kali |
| Puerto cliente | 33648 | Extremo temporal de esta conexión |
| IP servidor | 192.168.50.10 | Dirección de Ubuntu |
| Puerto servidor | 22 | Extremo de SSH |
| Protocolo | TCP | Transporte de esa conversación |

Para distinguir conversaciones considero ambos extremos y el protocolo. No basta con decir puerto 22: varios clientes o sesiones pueden usar el mismo puerto del servidor. Una regla basada en puertos tampoco garantiza por sí sola qué aplicación está transportando los bytes.

### Establecimiento

| Paso | Qué ocurre |
| --- | --- |
| SYN | El cliente propone el inicio y su número inicial de secuencia. |
| SYN/ACK | El servidor confirma el SYN y presenta su secuencia inicial. |
| ACK | El cliente confirma el SYN del servidor. |

La secuencia empieza con valores reales que no tienen por qué ser cero. Wireshark suele mostrar valores relativos para facilitar la lectura. El primer SYN puede verse como Seq=0; ese cero de la interfaz no implica que todos los equipos hayan elegido el mismo valor real.

### Seq, Ack y Len

Seq ubica bytes dentro del sentido de envío. Ack señala el siguiente número de secuencia esperado del otro extremo. Es una confirmación de transporte, no una respuesta de la aplicación ni una confirmación de autenticación. Len en TCP se refiere a datos TCP, no a toda la trama Ethernet.

SYN y FIN consumen una posición de secuencia. En mi cierre, el FIN de Kali con Seq relativo 16978 se confirma con Ack 16979 aunque no lleve datos de aplicación. El detalle explica por qué sumar únicamente Len no alcanza para todos los segmentos.

### Cierre y reinicio

FIN indica que un extremo terminó de enviar; el otro sentido puede seguir abierto hasta su propio cierre. En el PCAP 04 observé dos FIN y sus confirmaciones. RST indica una terminación o rechazo abrupto según el contexto; no es lo mismo que un cierre ordenado. No observé ese cierre abrupto como resultado principal del archivo 04.

Como ampliación, TIME_WAIT puede aparecer después del cierre y no significa necesariamente que una sesión siga autenticada. También existen retransmisiones y controles de ventana: una retransmisión aislada no demuestra un ataque. Los estudio en futuras capturas sin atribuirlos a resultados no documentados aquí.

## 6. SSH: cifrado e identidad en dos direcciones

SSH permite acceso remoto protegido. En mi laboratorio Kali inicia la conexión y Ubuntu ejecuta el servidor OpenSSH. Primero se establece TCP; luego SSH intercambia identificación, negocia mecanismos y establece el canal protegido, verifica la identidad del host y autentica la cuenta según el método permitido.

| Elemento | Qué protege o identifica | Ubicación o evidencia |
| --- | --- | --- |
| Clave del host | Identidad criptográfica del servidor | Archivos ssh_host_* en Ubuntu |
| Huella de clave pública | Resumen que permite comparar una clave | SHA256 mostrado por ssh y ssh-keygen |
| known_hosts | Claves de servidores conocidas por el cliente | Habitualmente ~/.ssh/known_hosts en Kali |
| Contraseña de maxi | Autenticación de la cuenta en esta práctica | Registro Accepted password |
| Clave de usuario | Método de autenticación alternativo | No lo configuré en este laboratorio |
| authorized_keys | Claves públicas autorizadas para una cuenta | Concepto complementario; no fue el método practicado |

Comparar la huella con la consola del servidor aporta una referencia independiente. Aceptar una huella desconocida sin compararla es una decisión de confianza inicial, no una verificación independiente de identidad. Si una clave cambia, antes de reemplazarla investigo reinstalaciones, cambios legítimos o posibles problemas.

Clave pública no significa clave privada. La pública puede distribuirse; la privada debe mantenerse protegida. La huella de una clave pública no permite iniciar sesión por sí sola. Un archivo con extensión .pub contiene material público; mostrar con ls -l el nombre de una clave privada y sus permisos tampoco revela su contenido.

### Qué puedo observar sin descifrar

En una captura puedo ver direcciones, puertos, tiempos y tamaños. Los banners iniciales y parte de la negociación SSH pueden ser visibles. Después no puedo leer automáticamente los comandos o contraseñas de la sesión cifrada. Follow TCP Stream no elimina el cifrado.

Por eso correlacioné fuentes: la red describe la conversación, ss su estado local y los registros de SSH el resultado de autenticación. La cuenta maxi es una identidad del sistema; no prueba por sí sola quién estaba físicamente frente al teclado.

### Ampliación para estudiar después

La autenticación por clave de usuario requiere un par propio, su privada protegida y una pública autorizada en el servidor. No necesito hacer ese cambio para entender la clave del host: resuelven identidades diferentes. Tampoco doy por hecho que deshabilitar contraseñas o cambiar el puerto sea parte del laboratorio completado.

## 7. Linux: programa, proceso, servicio y socket

Un programa es código y archivos instalados. Un proceso es una instancia en ejecución. Un servicio es una función administrada por el sistema. Un socket es un recurso de comunicación que un proceso o el administrador de servicios puede mantener abierto. Estas categorías están relacionadas, pero no son equivalentes.

| Observación | Lo que puedo afirmar |
| --- | --- |
| openssh-server instalado | Existen componentes del servidor; todavía debo revisar escucha y estado. |
| ssh.service active (running) | La unidad de servicio está ejecutándose. |
| ssh.socket active (listening) | Hay un socket administrado esperando actividad. |
| TCP LISTEN | Existe un extremo dispuesto a aceptar conexiones. |
| TCP ESTABLISHED | Hay una conexión TCP establecida en ese instante. |
| Accepted password | El servidor registró autenticación exitosa mediante contraseña. |

El primer estado que observé, servicio inactivo y socket activo, fue compatible con activación por socket. systemd podía iniciar el servicio al llegar una conexión. No necesité cambiarlo simplemente para que todas las líneas dijeran enabled. La escucha seguía expuesta a los enlaces permitidos: activación por socket no implica automáticamente menor superficie de ataque.

### Unidades y dependencias

systemd organiza unidades con relaciones y dependencias. Una .service describe un servicio; una .socket administra un socket para activación; una .timer programa una activación; una .target agrupa unidades para un estado u objetivo. Incluyo timer y target como contexto de estudio; no configuré unidades nuevas en esta práctica.

enabled se refiere a habilitación, mientras active describe estado actual. Un servicio disabled puede estar ejecutándose por inicio manual o activación de otra unidad. Una unidad enabled puede estar detenida. El preset representa una política predeterminada y no reemplaza la lectura de los estados reales.

### Registros y recursos

journald recoge entradas del journal y journalctl las consulta. No asumo que concentre absolutamente todos los logs de toda aplicación ni que retenga indefinidamente cada arranque. Un PID identifica un proceso en un momento; un fd identifica un descriptor abierto dentro de ese proceso; un puerto es un dato del extremo de red.

Memory, Tasks y CPU de systemctl describen contabilidad de la unidad según el sistema. No representan directamente toda la memoria de la VM, la cantidad de usuarios o un porcentaje de uso de CPU. Esta lectura evita interpretar cifras correctas con unidades equivocadas.

## 8. Usuarios, permisos y mínimo privilegio

Linux distingue identidades y permisos para controlar operaciones. UID identifica una cuenta; GID un grupo. La cuenta puede pertenecer a varios grupos. En mis pruebas fui maxi con UID 1000 y acceso a sudo, pero ejecuté la verificación de escritura sin elevar privilegios.

En -rw-------, el primer carácter identifica un archivo regular y luego aparecen tres grupos de permisos: propietario, grupo y otros. En un directorio el primer carácter es d. Uso r para lectura, w para escritura y x para ejecución o recorrido, según el tipo de objeto.

| Modo | Propietario | Grupo | Otros | Uso en el estudio |
| --- | --- | --- | --- | --- |
| 600: -rw------- | Leer y escribir | Sin permisos | Sin permisos | Estado final de nota.txt |
| 400: -r-------- | Leer | Sin permisos | Sin permisos | Archivo sin escritura del propietario |
| 644: -rw-r--r-- | Leer y escribir | Leer | Leer | Ejemplo habitual para datos públicos; no permiso universal |
| 700: drwx------ | Listar, modificar y atravesar | Sin permisos | Sin permisos | Ejemplo de directorio privado |

La notación octal asigna r=4, w=2 y x=1; sumo dentro de cada grupo. 600 se obtiene con 6 para propietario y 0 para grupo y otros. En el laboratorio modifiqué permisos con notación simbólica, no necesito atribuir un chmod 600 que no figura en el procedimiento.

### Archivos frente a directorios

Leer un archivo y listar un directorio son operaciones distintas. Para acceder a una ruta necesito permisos de recorrido en los directorios correspondientes. Para crear, renombrar o borrar entradas importan los permisos del directorio, además de restricciones como sticky bit, ACL u otras políticas. Estas últimas son ampliaciones; no las configuré aquí.

Eliminar escritura de un archivo no garantiza que no pueda borrarse su nombre desde un directorio modificable. Esta diferencia es relevante para proteger datos: una sola línea de chmod no describe toda la seguridad de una ruta.

### Qué comprobé con nota.txt

Partí de permisos que permitían escritura al propietario. Quité permisos de grupo y otros, luego escritura del propietario, y observé el error de la shell al abrir el archivo con >>. Al restaurar u+w pude agregar contenido. No usé una solución con root para ocultar el efecto del cambio.

Mínimo privilegio significa otorgar lo necesario para una tarea y evitar permisos que no aportan. En este ejemplo el archivo quedó accesible solo para su propietario. No implica que haya auditado todos los permisos del sistema ni que root haya perdido la capacidad de administrar el archivo.

Como ampliación, umask influye en los permisos iniciales al crear archivos y directorios. No es una orden que modifique retroactivamente cada archivo existente. Antes de repetir un ejercicio observo el estado inicial con ls -l en lugar de suponer que coincide con otra instalación.

## 9. Firewall, persistencia y diagnóstico con evidencia

Un firewall de host aplica filtrado en la propia máquina. UFW facilita administrar reglas de filtrado de Linux. En este laboratorio configuré UFW en Ubuntu; no configuré una regla equivalente en Kali ni en el router de la casa.

La dirección de una regla siempre necesita un punto de vista. La conexión se inicia desde Kali y llega a Ubuntu; por eso es saliente para Kali y entrante para Ubuntu. Mi regla permite entrada por enp0s8 desde 192.168.50.20 hacia 192.168.50.10, TCP 22.

| Concepto | Distinción que necesito recordar |
| --- | --- |
| Política predeterminada | Decisión general cuando el tráfico llega a ella sin coincidir antes con una regla aplicable. |
| Regla específica | Condiciones de interfaz, direcciones, protocolo y puerto para una acción. |
| Tráfico establecido | Puede admitirse por seguimiento de conexiones; una sesión anterior no valida necesariamente nuevas aperturas. |
| Reglas guardadas | Pueden existir aunque el firewall esté inactivo. |
| Estado activo | Debo consultarlo y relacionarlo con la configuración y las pruebas. |

No describo deny incoming como bloqueo literal de cada paquete entrante. El conjunto de UFW incluye reglas previas y tratamiento por protocolo. Un ping que siga respondiendo no demuestra por sí solo que UFW esté desactivado. De igual manera, permitir una IP no autentica una persona ni evita todas las formas de suplantación dentro del segmento.

### Persistencia en tres niveles

Para cada cambio distingo configuración guardada, configuración aplicada y funcionamiento comprobado. NetworkManager guarda perfiles; Netplan describe configuración que aplica un backend; UFW conserva reglas y habilitación. ip muestra el estado actual, no todos los archivos que lo originaron.

Un reinicio o apagado/arranque permite comprobar reconstrucción del estado. Guardar la VM y reanudarla conserva un estado suspendido: responde otra pregunta. Por eso no lo uso como sustituto de una prueba de persistencia.

### Hecho, explicación y límite

| Nivel | Ejemplo correcto |
| --- | --- |
| Hecho | Capturé tres solicitudes ARP por .99 sin respuesta dentro del archivo. |
| Interpretación | No se completó la resolución del vecino durante esa prueba. |
| Hipótesis | El destino podría estar apagado, desconectado o tener otra IP. |
| Dato faltante | Estado del equipo destino y su conexión al segmento. |

Con este método evito afirmar más de lo que las herramientas permiten. Mi objetivo no es que todos los comandos den una salida favorable, sino poder localizar el paso que falla y justificar el siguiente intento.

## 10. Repaso autónomo y próximos estudios

Uso estas preguntas abiertas después de leer el material. Intento responderlas sin volver a la tabla y luego comparo con las respuestas orientativas.

1. ¿Por qué una IP privada no identifica por sí sola el modo de red de VirtualBox?

2. ¿Por qué Kali resuelve la MAC de 10.0.2.2 para un ping a 8.8.8.8?

3. ¿Por qué una solicitud ARP contiene IPv4 sin ser un paquete IPv4?

4. ¿Cuántas tramas transportan cuatro solicitudes de ping y sus cuatro respuestas, sin fragmentación ni otro tráfico?

5. ¿Cómo puede ssh.service estar disabled e iniciar una sesión correctamente?

6. ¿Qué demuestra ESTABLISHED y qué dato adicional necesito para afirmar autenticación?

7. ¿Por qué una sesión SSH que sigue abierta no basta para validar una regla nueva?

8. ¿Qué diferencia hay entre la huella del host y la contraseña de maxi?

9. ¿Por qué quitar w a un archivo no resuelve por sí solo todas las posibilidades de borrado?

10. ¿Qué tres niveles reviso cuando una configuración desaparece después del arranque?

### Respuestas orientativas

| Pregunta | Respuesta |
| --- | --- |
| 1 | El modo lo define la conexión virtual; distintos modos pueden usar direcciones privadas. |
| 2 | La ruta selecciona ese gateway como siguiente salto local. La IP final del paquete sigue siendo 8.8.8.8. |
| 3 | Las IP son campos del mensaje ARP, transportado directamente por Ethernet. |
| 4 | Ocho: solicitud y respuesta son transmisiones distintas; las capas internas no se cuentan aparte. |
| 5 | disabled no impide otras activaciones; en mi caso ssh.socket estaba habilitado y escuchando. |
| 6 | Estado de la conexión TCP; necesito evidencia de autenticación, como el registro Accepted password. |
| 7 | El seguimiento de conexiones puede conservar tráfico establecido; pruebo una conexión nueva. |
| 8 | La huella permite comparar la clave pública del servidor; la contraseña autentica mi cuenta. |
| 9 | Borrar una entrada depende también de permisos del directorio y otras restricciones. |
| 10 | Lo guardado, lo aplicado al sistema y lo comprobado mediante una prueba real. |

### Ampliaciones que todavía no considero realizadas

Puedo continuar con DNS y DHCP observados activamente, claves SSH de usuario, permisos de directorios y umask, y pruebas negativas controladas de UFW. Después puedo relacionar procesos con conexiones y automatizar consultas con Python o Bash. Estos temas aprovechan la base del laboratorio y evitan convertir una lista de herramientas en un sustituto de los fundamentos.

### Fuentes de consulta

- ARP: https://www.rfc-editor.org/rfc/rfc826

- ICMP: https://www.rfc-editor.org/rfc/rfc792

- TCP: https://www.rfc-editor.org/rfc/rfc9293

- VirtualBox, manual de redes virtuales: https://www.virtualbox.org/manual/ch06.html

- OpenSSH, ssh y sshd: https://man.openbsd.org/ssh y https://man.openbsd.org/sshd

- systemd, unidades y sockets: https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html y https://www.freedesktop.org/software/systemd/man/latest/systemd.socket.html

- Netplan y NetworkManager: https://netplan.readthedocs.io/en/stable/ y https://networkmanager.dev/docs/api/latest/nmcli.html

- GNU Coreutils, permisos: https://www.gnu.org/software/coreutils/manual/coreutils.html

- UFW: https://manpages.ubuntu.com/manpages/noble/man8/ufw.8.html

Para repasar el proyecto vuelvo a cinco relaciones: dirección y ruta; vecino y trama; socket y servicio; identidad y permisos; regla y evidencia. Si puedo explicarlas y comprobarlas en las VM, puedo repetir el laboratorio con criterio propio.
