# HomeLab 01 — Comandos e interpretación

Adaptación para lectura en GitHub del [documento original](02_HomeLab01_Comandos.docx). Se conservan sus contenidos, tablas y figuras; la presentación y la paginación son distintas.

[Volver al proyecto](../README.md)

HOMELAB 01

Comandos y lectura de resultados

Referencia práctica para configurar, comprobar y diagnosticar

Máximo Rodríguez

Redes · Linux · Administración · Fundamentos de seguridad

Edición de estudio | 14 de noviembre de 2026

Documento 02 de 3

### Cómo uso este documento

Busco una tarea, identifico en qué VM debo ejecutarla y distingo consulta de modificación. Los valores de ejemplo pertenecen a este laboratorio.

### Contenido

1. Cómo leo y utilizo los comandos

2. Inventario de Linux, discos e identidad

3. Interfaces, rutas y vecinos

4. Configuración activa y persistente

5. Ping y filtros de Wireshark

6. SSH, systemctl y journalctl

7. Sockets y conexiones con ss

8. Archivos, permisos y verificación de escritura

9. UFW, persistencia y rutina de diagnóstico

10. Fuentes y criterios de reproducción

## 1. Cómo leo y utilizo los comandos

Uso este documento como referencia del laboratorio. Antes de ejecutar algo identifico el equipo, la cuenta, el objetivo y el efecto. Una terminal abierta en Kali puede estar ejecutando comandos en Ubuntu si estoy dentro de una sesión SSH. Compruebo whoami y hostname cuando tengo dudas.

Los comandos de consulta no equivalen a pruebas completas. Por ejemplo, ip route get informa una decisión local; ping genera tráfico y espera una respuesta; ss muestra sockets y journalctl muestra registros. Relaciono sus resultados para responder una pregunta concreta.

| Elemento de sintaxis | Cómo lo interpreto |
| --- | --- |
| sudo | Ejecución con privilegios permitidos por la política; no lo agrego automáticamente a todo. |
| ~ | Directorio personal del usuario según el contexto de la shell. En la sesión de maxi apunta a /home/maxi. |
| / | Raíz del sistema de archivos; una ruta que empieza por / es absoluta. |
| Comillas dobles | Mantienen unido un argumento con espacios, como Wired connection 2. |
| Barra inversa al final de línea | Continúa un comando en la siguiente línea en Bash; no dejo espacios después de ella. |
| >> | Agrega salida a un archivo; la shell abre el archivo para escribir. |
| > | Reemplaza el contenido de un archivo; no la uso cuando quiero conservar lo anterior. |
| # al inicio de una línea | Comentario explicativo, no una instrucción para ejecutar. |
| Opciones -n, -l o similares | Su significado depende del programa: -n no significa lo mismo en todas las herramientas. |

Las salidas de este documento son valores observados o descripciones de campos, no terminales simuladas. Cuando propongo una consulta complementaria la identifico como tal. Los nombres de interfaz y perfil de otra instalación pueden diferir.

### Consultar ayuda sin depender del chat

Como ampliación de estudio uso man ip, man nmcli, man ssh, man ss, man chmod y man ufw. Dentro de man busco con /palabra y salgo con q. Para una referencia corta reviso --help cuando el programa lo admite. Consulto la ayuda de la versión instalada antes de adaptar opciones nuevas.

## 2. Inventario de Linux, discos e identidad

cat /etc/os-release

lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS

whoami

hostname

id

| Comando | Qué responde | Qué no demuestra |
| --- | --- | --- |
| cat /etc/os-release | Identificación declarada de la distribución y versión | No enumera por sí solo todos los paquetes ni la versión exacta de cada herramienta. |
| lsblk -o ... | Dispositivos de bloques y columnas seleccionadas | SIZE es capacidad; no espacio libre del sistema de archivos. |
| whoami | Nombre asociado a mi identidad efectiva | No identifica por sí solo el equipo. |
| hostname | Nombre configurado del sistema donde se ejecuta | No prueba resolución DNS ni pertenencia a un dominio. |
| id | UID, GID y grupos de la cuenta | Pertenecer a sudo no significa que cada comando se ejecute como root. |
| Campo de lsblk | Dato del laboratorio | Interpretación |
| NAME | sda, sda1, sda2 | Nombres del disco y sus particiones; la numeración puede cambiar. |
| SIZE | sda 25G | Capacidad presentada del disco virtual. |
| TYPE | disk, part, loop, rom | Tipo de dispositivo de bloques. |
| MOUNTPOINTS | / en sda2 | Sistema de archivos usado como raíz. |
| loop0 y otros | Dispositivos loop | Imágenes de archivos montadas; en las notas se relacionan con paquetes Snap. |
| sr0 | rom | Unidad óptica virtual. |

En id observé uid=1000(maxi), gid=1000(maxi) y el grupo 27(sudo), además del grupo principal. UID 0 corresponde a root; 1000 es un identificador local y no una regla universal para todas las cuentas. En otra instalación los grupos pueden variar.

Como consulta adicional para diferenciar capacidad y espacio libre puedo usar df -h /. No la presento como una salida conservada del laboratorio.

## 3. Interfaces, rutas y vecinos

ip -br address

ip -4 address show dev eth1

ip route

ip route get 192.168.50.10

ip neigh show to 192.168.50.10 dev eth1

En estos ejemplos de Kali, -br pide una vista breve; -4 limita la consulta a IPv4; dev identifica la interfaz. En Ubuntu adapto eth1 a enp0s8 cuando corresponde. No cambio el nombre de una interfaz solamente para que coincida con la guía.

| Dato | Lectura útil |
| --- | --- |
| lo, 127.0.0.1/8 y ::1/128 | Interfaz loopback y direcciones de comunicación con el propio sistema. |
| UP | Estado mostrado como activo; no garantiza respuesta de un destino remoto. |
| UNKNOWN en lo | Es habitual en loopback y no significa por sí solo un fallo. |
| fe80::… | IPv6 de enlace local; no prueba que exista IPv4 configurada. |
| 192.168.50.20/24 | Dirección de Kali y prefijo de 24 bits; red 192.168.50.0/24. |
| docker0 DOWN | Estado del puente observado; no concluyo que todo el servicio Docker esté detenido. |

### Cómo descompongo una ruta

default via 10.0.2.2 dev eth0 proto static metric 100

10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.50 metric 100

| Campo | Interpretación |
| --- | --- |
| default | Ruta para destinos sin una coincidencia más específica; equivale a /0. |
| via | Siguiente salto; para la salida NAT es 10.0.2.2. |
| dev | Interfaz de salida seleccionada. |
| proto static | Origen o clasificación de la ruta; no demuestra que la haya escrito manualmente. |
| proto kernel | Ruta instalada por el kernel, habitualmente por una dirección de interfaz. |
| scope link | Destino alcanzable directamente por el enlace. |
| src | Dirección de origen preferida para tráfico local que use esa ruta. |
| metric | Preferencia entre rutas comparables; no desplaza por sí sola una coincidencia más específica. |
| linkdown | La ruta se relaciona con un enlace inactivo. |

No confundo una ruta a una red con una conexión activa. La ausencia de via en una ruta directamente conectada indica que no se necesita un router intermedio para ese destino. En una red con políticas de enrutamiento el análisis puede requerir más tablas y reglas; el laboratorio utiliza el caso básico mostrado.

### Vecinos IPv4 y ARP

192.168.50.10 lladdr 08:00:27:b6:92:36 STALE

| Campo o estado | Significado |
| --- | --- |
| Dirección inicial | IPv4 del vecino consultado. |
| lladdr | Dirección de enlace asociada, aquí la MAC Ethernet. |
| REACHABLE | Alcanzabilidad confirmada recientemente. |
| STALE | La asociación se conserva, pero necesita revalidación de alcanzabilidad según el uso y los temporizadores. |
| INCOMPLETE | Resolución en curso, sin asociación completada todavía. |
| FAILED | La resolución o comprobación del vecino no se completó. |

REACHABLE e INCOMPLETE se incluyen como ampliación para interpretar futuras consultas; STALE y FAILED aparecen en el material del laboratorio. La caché es temporal y puede cambiar entre consultas.

sudo ip neigh del 192.168.50.10 dev eth1

Esta modificación elimina solo la entrada indicada. La utilicé para provocar una nueva resolución ARP bajo captura. Si la entrada no existe, no lo interpreto automáticamente como fallo de la red.

## 4. Configuración activa y persistente

### ip address: cambio inmediato

# Ubuntu

sudo ip address add 192.168.50.10/24 dev enp0s8

# Kali

sudo ip address add 192.168.50.20/24 dev eth1

add agrega una dirección; no modifica el perfil persistente de NetworkManager ni escribe Netplan. Si la IP ya existe, repetirlo puede producir un error de duplicación. La configuración persistente es la ruta normal para repetir el estado final del laboratorio.

### NetworkManager en Kali

nmcli -f NAME,TYPE,DEVICE connection show

nmcli -f GENERAL,IP4 device show eth1

sudo nmcli connection modify "Wired connection 2" \

  ipv4.method manual ipv4.addresses 192.168.50.20/24 \

  ipv4.gateway "" ipv4.never-default yes \

  connection.autoconnect yes

sudo nmcli connection up "Wired connection 2"

| Parte | Qué hago con ella |
| --- | --- |
| -f NAME,TYPE,DEVICE | Selecciono columnas para identificar perfiles y dispositivos. |
| connection show | Consulto perfiles guardados y su asociación visible con dispositivos. |
| device show eth1 | Consulto información del dispositivo y su configuración activa. |
| connection modify | Modifico el perfil guardado. |
| ipv4.method manual | Uso IPv4 fija en este perfil. |
| ipv4.addresses | Establezco dirección y prefijo. |
| ipv4.gateway "" | Dejo vacío el gateway del perfil interno. |
| ipv4.never-default yes | Impido que este perfil aporte una ruta IPv4 predeterminada. |
| connection.autoconnect yes | Permito activación automática del perfil cuando corresponda. |
| connection up | Activo el perfil; compruebo después qué quedó aplicado. |

En device show, GENERAL.STATE describe el estado gestionado, GENERAL.CONNECTION el perfil asociado, GENERAL.HWADDR la MAC y IP4.ADDRESS la IPv4 activa. UUID identifica un perfil; no es una IP, una MAC ni un secreto de autenticación. Estos campos ayudan a explicar por qué modificar solo el kernel no bastó.

### Netplan en Ubuntu

ls -l /etc/netplan/

sudo netplan get

sudo cp -p -n /etc/netplan/00-installer-config.yaml \

  /etc/netplan/00-installer-config.yaml.bak

sudo nano /etc/netplan/00-installer-config.yaml

sudo netplan generate

sudo netplan try

| Acción | Efecto y comprobación |
| --- | --- |
| ls -l | Identifico nombres y permisos, sin leer todavía toda la configuración. |
| netplan get | Leo la configuración combinada declarada en los archivos. |
| cp -p -n | Copio preservando atributos cuando es posible y sin sobrescribir un respaldo existente; compruebo que se creó. |
| nano | Edito YAML: uso espacios y respeto la jerarquía network → ethernets → interfaz. |
| generate | Valido y genero archivos para el backend; una salida vacía no sustituye la revisión de errores y del estado. |
| try | Aplico temporalmente y confirmo después de probar; si vence o falla, reviso estado y archivos antes de continuar. |

En nano guardo con Ctrl+O, confirmo el nombre con Enter y salgo con Ctrl+X. La guía principal incluye el YAML completo documentado. La MAC del adaptador NAT debe pertenecer a la VM correspondiente; no la copio sin comprobarla.

No deduzco el backend solo por el nombre Ubuntu Server. Netplan puede generar configuración para networkd o NetworkManager según lo declarado y la instalación. En este laboratorio distinguí su función de la herramienta que muestra el estado activo, ip.

## 5. Ping y filtros de Wireshark

ping -c 4 192.168.50.10

ping -c 4 8.8.8.8

ping -c 3 -W 1 192.168.50.99

| Dato o argumento | Cómo lo interpreto |
| --- | --- |
| -c 4 | Límite de cuatro solicitudes. |
| -W 1 | Espera de respuesta configurada en segundos; no interpreto que toda la ejecución deba durar un segundo. |
| icmp_seq | Número de secuencia que permite relacionar la respuesta con una solicitud. |
| ttl | TTL restante en el paquete recibido; no es el tiempo de viaje. |
| time | Tiempo de ida y vuelta observado por ping, normalmente en milisegundos. |
| packet loss | Proporción de solicitudes sin respuesta contabilizada por esa ejecución. |
| rtt min/avg/max/mdev | Resumen de los tiempos medidos; mdev refleja su dispersión, no ancho de banda. |
| Destination Host Unreachable | Mensaje de error cuyo emisor y contexto debo identificar; en la prueba local hubo fallo de resolución ARP. |

Que ping responda demuestra un intercambio ICMP, no que SSH esté instalado ni que todos los protocolos estén permitidos. Si falla el ping, un firewall que trate ICMP de otra forma es una posibilidad; no concluyo automáticamente que el host esté apagado.

### Filtros de visualización usados

| Filtro | Uso |
| --- | --- |
| arp \|\| icmp | ARP o ICMP en la interfaz elegida. |
| tcp.port == 22 | Paquetes TCP con puerto 22 en cualquiera de los extremos. |
| tcp.port == 22 && tcp.port == 33648 | Conversación del PCAP 04 por sus dos puertos, dentro del contexto de esa captura. |
| tcp.port == 22 && (tcp.flags.fin == 1 \|\| tcp.flags.reset == 1) | Busco segmentos de cierre o reinicio; oculta ACK sin esos flags. |

Para copiar los filtros sin escapes de tabla:

arp || icmp

tcp.port == 22

tcp.port == 22 && tcp.port == 33648

tcp.port == 22 && (tcp.flags.fin == 1 || tcp.flags.reset == 1)

Como ampliación de reproducción puedo seleccionar la conversación en Wireshark y usar tcp.stream == N, reemplazando N por el índice asignado en ese archivo. No invento un índice universal. Un puerto aislado tampoco identifica de manera única todas las sesiones de un archivo grande.

### Campos que necesito localizar

| Bloque | Campos relevantes | Pregunta que responden |
| --- | --- | --- |
| Frame | Número y tiempo de captura | ¿Dónde encuentro la evidencia en este archivo? |
| Ethernet II | Source, Destination, Type | ¿Qué interfaz envía y quién recibe en el enlace? |
| ARP | Opcode, Sender MAC/IP, Target MAC/IP | ¿Qué asociación se pregunta o anuncia? |
| IPv4 | Source, Destination, TTL, Protocol | ¿Cuál es el destino IP y qué transporta el paquete? |
| ICMP | Type, Identifier, Sequence Number | ¿Es solicitud o respuesta y con cuál se relaciona? |
| TCP | Puertos, Seq, Ack, flags y Len | ¿Qué extremos participan y qué parte del flujo se envía o confirma? |

La columna Protocol resume una capa reconocida: una fila rotulada SSH sigue teniendo TCP, IP y Ethernet. El número de bloques desplegables no multiplica el número de tramas. Los tiempos relativos dependen de la visualización; para correlacionar registros necesito conocer también zona horaria y reloj.

## 6. SSH, systemctl y journalctl

# Ubuntu

sudo apt update

sudo apt install openssh-server

systemctl status ssh.service ssh.socket --no-pager

sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub

# Kali

ssh maxi@192.168.50.10

En ssh, maxi es la cuenta remota y 192.168.50.10 el servidor. La contraseña se introduce en el diálogo de autenticación y normalmente no se muestra mientras escribo. ssh-keygen -l muestra una huella y -f selecciona el archivo; en este comando no estoy generando una nueva clave de usuario.

### Interpretación de systemctl status

| Dato de la salida | Significado |
| --- | --- |
| Loaded: loaded | systemd cargó la definición de la unidad. |
| /usr/lib/systemd/system/ssh.service | Ruta de la definición suministrada por el sistema o paquete. |
| enabled / disabled | Estado de habilitación para arranque o activación mediante enlaces; es distinto del estado actual. |
| preset: enabled | Política predeterminada recomendada, no garantía de la configuración actual. |
| Active: inactive (dead) | Unidad detenida en ese instante; en el caso observado existía activación por socket. |
| Active: active (running) | Servicio en ejecución. |
| Active: active (listening) | Socket activo y esperando conexiones. |
| TriggeredBy / Triggers | Relación de activación entre socket y servicio. |
| Listen: 0.0.0.0:22 / [::]:22 | Direcciones de escucha para IPv4 / IPv6. |
| Main PID | Identificador del proceso principal; puede cambiar al reiniciar. |
| Tasks | Procesos e hilos contabilizados en el grupo de la unidad, no usuarios conectados. |
| Memory / peak | Memoria contabilizada para la unidad y máximo registrado; no RAM total del equipo. |
| CPU | Tiempo acumulado de CPU; no porcentaje instantáneo. |
| CGroup | Grupo de control donde se organizan los procesos y recursos. |
| Invocation | Identificador de una activación de la unidad; no identidad de una sesión SSH. |
| Docs: man:sshd(8) | Referencia al manual; 8 es la sección. |
| --no-pager | Muestro la salida sin abrir un paginador. |

### Registros del servicio

sudo journalctl -u ssh.service -b -n 30 --no-pager

| Parte | Qué selecciona |
| --- | --- |
| -u ssh.service | Entradas asociadas a la unidad indicada. |
| -b | Arranque actual. |
| -n 30 | Hasta las últimas 30 entradas que cumplen la consulta. |
| --no-pager | Salida directa. |

En el laboratorio relacioné la hora, el proceso, la cuenta, la IP y el puerto. Accepted password significa autenticación con contraseña aceptada; Failed password indica rechazo; session opened y session closed registran apertura y cierre de sesión. Un timeout antes de autenticar no prueba por sí mismo un ataque.

El journal no garantiza conservar indefinidamente todos los arranques: depende de su configuración y retención. Como ampliación, -b -1 consulta el arranque anterior si existen sus registros. Una consulta vacía puede deberse al filtro, los permisos o la retención; no equivale automáticamente a ausencia de actividad.

## 7. Sockets y conexiones con ss

sudo ss -lntp

sudo ss -ntp state established

| Opción | Función |
| --- | --- |
| -l | Sockets en escucha. |
| -n | Direcciones y puertos numéricos. |
| -t | TCP. |
| -p | Procesos asociados cuando los permisos permiten verlos. |
| state established | Conexiones en ese estado TCP. |
| sudo | Facilita ver procesos de otros usuarios; no lo necesito para generar un ping normal. |

### Columnas: el estado cambia el significado

| Campo | En LISTEN | En una conexión TCP establecida |
| --- | --- | --- |
| Recv-Q | Conexiones completadas pendientes de aceptación | Bytes recibidos pendientes de lectura por la aplicación |
| Send-Q | Límite mostrado para la cola de aceptación | Bytes enviados pendientes de confirmación, en la lectura habitual de TCP |
| Local Address:Port | Dirección de escucha local | Extremo local de esa conexión |
| Peer Address:Port | Sin cliente concreto, puede mostrar comodines | Extremo remoto concreto |
| Process | Proceso que mantiene el socket | Proceso o procesos asociados a la conexión |

Por eso no interpreté Send-Q 4096 de LISTEN como 4096 bytes enviados. El estado y el protocolo importan. En la salida del laboratorio systemd tenía abierto el socket antes de que el servicio se activara; después aparecían procesos de SSH.

| Dato observado | Interpretación |
| --- | --- |
| 0.0.0.0:22 | Escucha en todas las direcciones IPv4 locales. |
| [::]:22 | Escucha IPv6; no deduzco el detalle de doble pila solo por esa línea. |
| 127.0.0.53%lo:53 y 127.0.0.54:53 | Puntos locales del resolvedor DNS observados; no son el destino de la sesión SSH. |
| pid=1 | Identificador del proceso systemd en esa salida. |
| fd=81 / fd=82 | Descriptores abiertos dentro de un proceso; no números de puerto. |
| 192.168.50.10:22 ↔ 192.168.50.20:50332 | Extremos de la sesión correlacionada con el registro. |

Una salida ESTABLISHED prueba estado TCP en ese momento, no necesariamente que la cuenta ya se haya autenticado. Para eso consulté también los registros de SSH.

## 8. Archivos, permisos y verificación de escritura

ls -ld /home/maxi /etc/ssh

ls -l /etc/ssh/ssh_host_ed25519_key \

  /etc/ssh/ssh_host_ed25519_key.pub

mkdir -p ~/homelab01/permisos

touch ~/homelab01/permisos/nota.txt

ls -l ~/homelab01/permisos/nota.txt

Con -d consulto el directorio mismo, no su contenido. mkdir -p crea los directorios que falten y admite que ya existan. touch crea un archivo vacío si no existe; si existe actualiza marcas temporales y no lo vacía.

| Columna de ls -l | Ejemplo del laboratorio | Lectura |
| --- | --- | --- |
| Tipo y permisos | -rw------- | Archivo regular; propietario lee y escribe; grupo y otros sin permisos. |
| Enlaces | 1 | Cantidad de enlaces duros indicada. |
| Propietario | maxi | Cuenta propietaria. |
| Grupo | maxi | Grupo propietario; no lista todos los grupos de la cuenta. |
| Tamaño | 71 | Bytes del archivo en la verificación posterior. |
| Fecha y hora | Nov 14 05:32 en la captura | Marca mostrada según la configuración local; no doy por hecho que coincida con mi hora de envío. |
| Nombre | nota.txt o ruta completa | Archivo consultado. |

chmod go-rw ~/homelab01/permisos/nota.txt

chmod u-w ~/homelab01/permisos/nota.txt

echo "Prueba de permisos" >> ~/homelab01/permisos/nota.txt

chmod u+w ~/homelab01/permisos/nota.txt

echo "Prueba de permisos" >> ~/homelab01/permisos/nota.txt

cat ~/homelab01/permisos/nota.txt

| Expresión | Cambio |
| --- | --- |
| go-rw | Quito lectura y escritura a grupo y otros. |
| u-w | Quito escritura al propietario. |
| u+w | Restauro escritura al propietario. |
| >> | Intento agregar una línea; falla si la shell no puede abrir el archivo para escritura. |

El error Permission denied fue esperado al quitar u-w. No ejecuté la prueba con sudo. Como ampliación, sudo echo ... >> archivo no eleva automáticamente la redirección: la shell que interpreta >> sigue siendo la que abre el archivo. No utilicé esa variante para resolver el ejercicio.

| Objeto | r | w | x |
| --- | --- | --- | --- |
| Archivo regular | Leer contenido | Modificar contenido | Ejecutarlo, sujeto a formato y otras restricciones |
| Directorio | Listar nombres | Modificar entradas, normalmente junto con x | Atravesar el directorio y acceder a entradas conocidas |

El permiso de borrar un archivo depende del directorio que contiene su entrada y de otras restricciones; no se determina únicamente por el bit w del propio archivo. Esta diferencia se amplía en el documento 03.

## 9. UFW, persistencia y rutina de diagnóstico

# Ubuntu

sudo ufw status verbose

sudo ufw show added

sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow in on enp0s8 proto tcp \

  from 192.168.50.20 to 192.168.50.10 port 22

sudo ufw enable

| Parte | Condición o efecto |
| --- | --- |
| status verbose | Estado actual, políticas y reglas mostradas por UFW. |
| show added | Reglas de usuario guardadas; pueden existir con UFW inactivo. |
| default deny incoming | Política predeterminada de entrada, sujeta al conjunto de reglas aplicables. |
| default allow outgoing | Permito conexiones iniciadas por Ubuntu. |
| allow in | Permiso de entrada desde la perspectiva de Ubuntu. |
| on enp0s8 | Interfaz por la que debe ingresar el tráfico. |
| proto tcp | Protocolo TCP. |
| from 192.168.50.20 | Dirección origen de Kali en el laboratorio. |
| to 192.168.50.10 port 22 | Dirección destino interna y puerto del servidor SSH. |
| enable | Activo UFW y su carga al iniciar el sistema. |

Antes de activar las reglas conservé una consola de Ubuntu accesible y definí el permiso SSH. Después comprobé una conexión nueva desde Kali. No confundí dejar funcionando una sesión anterior con validar una nueva.

# Cierro una sesión remota

exit

# Apago cada VM desde su consola cuando corresponde

sudo poweroff

poweroff apaga el sistema; reboot lo reinicia. Guardar el estado de una VM es otra operación y no prueba que la configuración se reconstruya al arrancar. Después del arranque consulto direcciones, rutas, UFW y una sesión SSH nueva. No repito comandos de modificación si el estado final ya es correcto.

### Orden de diagnóstico que puedo reutilizar

| Pregunta | Consulta o comprobación | Qué decido después |
| --- | --- | --- |
| ¿Estoy en el equipo correcto? | whoami y hostname | Identifico si la terminal es local o remota. |
| ¿Existe el enlace previsto? | Configuración VirtualBox e ip -br address | Reviso adaptador, cable, segmento y dirección. |
| ¿Se elige la salida correcta? | ip route get destino | Distingo entrega directa de gateway. |
| ¿Se resolvió el vecino? | ip neigh y captura ARP | Localizo un problema de enlace o resolución. |
| ¿Hay un servicio escuchando? | systemctl status y ss -lntp | Distingo red disponible de aplicación disponible. |
| ¿El firewall permite la prueba? | ufw status verbose y nueva conexión | Reviso dirección, interfaz, protocolo y puerto. |
| ¿Se autenticó la cuenta? | journalctl del servicio | Distingo fallo TCP de rechazo de autenticación. |
| ¿El cambio sobrevive al arranque? | Apagado/arranque y nuevas consultas | Comparo estado guardado, aplicado y funcional. |

## 10. Fuentes y criterios de reproducción

Para entender opciones consulto la documentación oficial y el manual instalado. Los valores concretos de salidas proceden de mis capturas y archivos, no de una salida genérica de Internet.

- NetworkManager, nmcli: https://networkmanager.dev/docs/api/latest/nmcli.html

- Netplan, configuración y comandos: https://netplan.readthedocs.io/en/stable/

- systemctl: https://www.freedesktop.org/software/systemd/man/latest/systemctl.html

- journalctl: https://www.freedesktop.org/software/systemd/man/latest/journalctl.html

- ss: https://man7.org/linux/man-pages/man8/ss.8.html

- OpenSSH: https://man.openbsd.org/ssh y https://man.openbsd.org/ssh-keygen

- UFW: https://manpages.ubuntu.com/manpages/noble/man8/ufw.8.html

- GNU Coreutils, chmod y permisos: https://www.gnu.org/software/coreutils/manual/coreutils.html

- Wireshark, guía de usuario: https://www.wireshark.org/docs/wsug_html_chunked/

No necesito obtener los mismos PID, puertos temporales, tiempos de ping o números de secuencia de una captura anterior. Necesito comprobar la misma relación: interfaz correcta, destino correcto, resultado esperado y una explicación compatible con la evidencia.
