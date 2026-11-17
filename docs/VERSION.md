# Registro de versión de HomeLab 01

Preparación para GitHub: 17 de noviembre de 2026.

## Documentos de referencia

Los tres DOCX y sus PDF corresponden a los archivos existentes en `docs/`, edición de estudio del 14 de noviembre de 2026. Se conservaron sin editar. Las versiones Markdown se adaptaron de los DOCX, manteniendo texto, tablas y figuras; no se presupone igualdad de paginación entre formatos.

| Archivo | SHA-256 |
| --- | --- |
| [01_HomeLab01_Guia_Informe.docx](01_HomeLab01_Guia_Informe.docx) | `e3f9f41ed091d585bbc29b5b61e3cef4c96a5bb9d518323f111b740369d7bf40` |
| [02_HomeLab01_Comandos.docx](02_HomeLab01_Comandos.docx) | `4f5114d6e6a2d085a4ffcd32b72907421c633c26f7d599a242b2fe78762d3afd` |
| [03_HomeLab01_Fundamentos.docx](03_HomeLab01_Fundamentos.docx) | `514a99e8e741a1e99965a9201dbebc554292ac165ac623de7ebcbc0b90622530` |
| [01_HomeLab01_Guia_Informe.pdf](01_HomeLab01_Guia_Informe.pdf) | `6f60d6bc5bc3d3823afd535ef796af0f82bda465a6a1c059e66a8c1cfdf521e9` |
| [02_HomeLab01_Comandos.pdf](02_HomeLab01_Comandos.pdf) | `a46c7923b0afe6e3afeff350fc0c10aeb8f07ffcbc7aa1fdf20745c0f1b21871` |
| [03_HomeLab01_Fundamentos.pdf](03_HomeLab01_Fundamentos.pdf) | `d6ba3fd29b00d8bebc4fd783907bff95c065c346357f7edb99346f802ca1b88a` |

## Adaptaciones y evidencias

- `informe.md`: guía e informe, con sus 17 figuras extraídas del DOCX.
- `comandos.md`: comandos e interpretación.
- `fundamentos.md`: fundamentos, glosario y preguntas de repaso.
- Los cuatro PCAPNG se conservaron sin editar. Sus huellas están en [SHA256SUMS](../evidencias/SHA256SUMS).

Los apuntes anteriores `Homelab 01.docx` y `HomeLab01.pdf`, en la raíz local, se conservan fuera de la publicación mediante `.gitignore`. El repositorio utiliza la edición organizada de `docs/`.

Los resultados se contrastaron con los documentos y los puntos recuperables de las conversaciones del proyecto. Esa recuperación no constituye una transcripción completa. No se ejecutaron nuevamente las pruebas en las VM durante la preparación para GitHub.

Si se modifica un documento de referencia, debe revisarse su adaptación Markdown y actualizarse esta huella. Los hashes identifican versiones; no prueban por sí solos la veracidad de los resultados.
