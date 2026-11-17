# Evidencias de HomeLab 01

[Volver al proyecto](../README.md)

Los nombres y números de trama se refieren a los archivos completos conservados aquí. Un filtro de visualización no cambia sus índices; exportar una selección a otro archivo puede cambiarlos.

| Archivo | Filtro de visualización | Referencias verificables |
| --- | --- | --- |
| [01-arp-icmp-red-interna.pcapng](01-arp-icmp-red-interna.pcapng) | `arp \|\| icmp` | 1–2: ARP por Ubuntu; 3–10: cuatro pares ICMP; 11–12: otro intercambio ARP |
| [02-icmp-destino-externo.pcapng](02-icmp-destino-externo.pcapng) | `arp \|\| icmp` | ARP por `10.0.2.2` y cuatro pares ICMP entre `10.0.2.50` y `8.8.8.8` |
| [03-arp-sin-respuesta.pcapng](03-arp-sin-respuesta.pcapng) | `arp` | Tres solicitudes por `192.168.50.99`, sin respuesta en este archivo |
| [04-ssh-conexion-y-cierre.pcapng](04-ssh-conexion-y-cierre.pcapng) | `tcp.port == 22` | 1–3: SYN, SYN/ACK, ACK; 775–778: FIN/ACK, ACK, FIN/ACK, ACK |

El archivo 01 contiene 15 tramas: 4 ARP, 8 ICMP y 3 IPv6. El archivo 02 contiene 17: 2 ARP, 8 ICMP y 7 IPv6. El archivo 03 contiene 3 ARP; el 04 contiene 778 TCP entre Kali y Ubuntu, con puerto cliente 33648 y servidor 22. Los conteos y extremos fueron comprobados nuevamente durante la preparación del repositorio.

## Integridad

Las huellas de [SHA256SUMS](SHA256SUMS) se calcularon nuevamente el 17 de noviembre de 2026 y coinciden con las documentadas en el informe. No se presentan como un registro de adquisición original ni como prueba de autoría por sí solas.

Desde esta carpeta, en Linux:

```bash
sha256sum -c SHA256SUMS
```

Desde la raíz del proyecto, en PowerShell:

```powershell
Get-FileHash .\evidencias\*.pcapng -Algorithm SHA256
```

En PowerShell se comparan los resultados con el manifiesto; `Get-FileHash` por sí solo no hace esa comparación.

## Figuras y límites

La carpeta [capturas](capturas/) contiene 17 figuras extraídas sin recomprimir del DOCX de la guía: `informe-01.png` es la topología reconstruida; las siguientes corresponden a las figuras del recorrido práctico. Consultar sus pies en [informe.md](../docs/informe.md).

Las capturas de pantalla y los PCAP no siempre corresponden a la misma sesión SSH: aparecen puertos cliente 50332, 41500 y 33648 en fuentes diferentes. Un PCAP cifrado no revela los comandos ni las contraseñas de SSH. Las direcciones privadas y MAC virtuales se conservan para seguir el razonamiento de red.
