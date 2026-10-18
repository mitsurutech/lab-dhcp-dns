# Laboratorio de DHCP y DNS con dnsmasq: un laboratorio de redes para clase, en una sola máquina virtual

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml)

Un **laboratorio de DHCP y DNS** listo para usar para enseñar cómo una red reparte direcciones y responde a nombres, de la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning), donde las escuelas y los centros dan a cada alumno su propia máquina virtual Ubuntu con una terminal en el navegador. Cada alumno recibe una pequeña red construida dentro de su máquina: un servidor y dos clientes, `pc1` y `pc2`, que arrancan **sin ninguna dirección**. El alumno configura **dnsmasq**, el pequeño servidor DHCP y DNS que se encuentra en los routers domésticos y en laboratorios de todas partes, para que el servidor ceda direcciones de un rango, diga a los clientes cuál es su router y su servidor DNS, dé a un cliente una **dirección fija según su dirección MAC** y responda por los nombres propios del laboratorio. No se instala nada en el ordenador del alumno.

El servidor y los clientes son **espacios de nombres de red** en un conmutador virtual, y las herramientas reales hacen el trabajo: `dnsmasq` en el servidor, `dhclient` en los clientes, `dig`, `ip` y `tcpdump` para observarlo. Ese es el objetivo.

## El laboratorio

| Laboratorio | Archivo | Qué hace el alumno | Página de ayuda |
| --- | --- | --- | --- |
| DHCP y DNS | [`dhcp-dns.lab.yaml`](dhcp-dns.lab.yaml) | Escribir la configuración de dnsmasq para que los clientes reciban direcciones de un rango con el router y el servidor DNS correctos, un cliente reciba siempre la misma dirección, y los dos nombres del laboratorio se resuelvan y ningún otro | [Laboratorio de DHCP y DNS](https://mitsurutech.co.uk/es/help/lab-dhcp-dns) |

El laboratorio deja en la carpeta personal del alumno un README que dice qué hacer y no da ninguna de las respuestas. Tiene una **comprobación**, un script que califica una máquina virtual entregada sin que el profesor tenga que leerla. Se ejecuta sobre una copia de la máquina guardada tras un arranque limpio, detiene el dnsmasq que estuviera en marcha, inicia uno a partir de los archivos guardados en `/etc/dnsmasq.d`, borra los dos clientes y hace que pidan de nuevo. Así que solo pasa si el comportamiento real es el correcto: los clientes reciben direcciones del rango con el router, el servidor DNS y la duración de cesión correctos, el segundo cliente recibe su dirección fija, los dos nombres se resuelven a las direcciones correctas y un nombre desconocido no recibe respuesta. Las direcciones escritas a mano en los clientes, una respuesta comodín para todos los nombres o nombres puestos en `/etc/hosts` no pasan.

## Qué hay en el archivo

`dhcp-dns.lab.yaml` es una **definición de laboratorio de máquina virtual** completa, exactamente lo que ejecuta la plataforma:

| Clave | Qué es |
| --- | --- |
| `cpus`, `memory`, `disk` | el tamaño de la máquina virtual de cada alumno: 1 CPU, 1024 MB y 10 GB |
| `open` | los puertos que se abren en el navegador: ninguno, el laboratorio es la terminal |
| `setup` | un script que se ejecuta una vez, como root, la primera vez que arranca la máquina virtual de cada alumno: instala dnsmasq, el cliente DHCP, `dig` y `tcpdump` desde el archivo de Ubuntu, escribe el comando `dnslab`, el README del laboratorio y la unidad que reconstruye la red al arrancar |
| `check` | el script de calificación, que se ejecuta como root sobre una copia del disco guardado tras un arranque limpio; su estado de salida es el resultado y `0` es aprobado |

El comando `dnslab` que instala: `dnslab status` lista las máquinas, `dnslab restart` vuelve a ejecutar dnsmasq desde la configuración guardada, `dnslab renew pc1` hace que un cliente pida una cesión y `dnslab shell pc1` abre una shell dentro de una máquina.

## Usarlo en la plataforma

**Para ponerlo como trabajo a una clase:**

1. En tu módulo, añade una tarea y elige la tarjeta **Laboratorio de VM Ubuntu**.
2. Pulsa el ejemplo, o pega el contenido del archivo de aquí. Usar un ejemplo sustituye el texto, y Deshacer recupera el tuyo.
3. Publica y prueba primero tu propia copia. Cada alumno recibe una máquina virtual propia, que se conserva entre sesiones.
4. Asígnala como cualquier tarea. Cuando se entregue, ejecuta la comprobación en la fila de un alumno de la lista de la clase: inicia una copia de su máquina guardada que no se guarda de vuelta y muestra si pasó y qué imprimió.

La [guía de laboratorios de VM Ubuntu](https://mitsurutech.co.uk/es/help/labs) del manual y su [página de laboratorios de ejemplo](https://mitsurutech.co.uk/es/help/sample-labs) cubren el resto, y una lección básica de este laboratorio, **DHCP and DNS Basics**, está en el Public college y en cada centro nuevo. Para cambiar el laboratorio, copia el archivo y edita juntas las direcciones y los nombres en `setup` y en `check`.

¿Aún no tienes una cuenta de centro? [El primer año es gratis](https://mitsurutech.co.uk/es/pricing) hasta 50 alumnos, y [registrarse](https://mitsurutech.co.uk/es/signup) solo pide una dirección de correo electrónico.

## Comprobar el archivo

En cada push, `.github/workflows/check.yml` comprueba que el archivo es un YAML válido con las cinco claves de arriba y que `setup` y `check` caben cada uno en el límite de 16 KB de la plataforma. No arranca una máquina virtual: eso se hace en las propias máquinas de la plataforma.

## Lo que no funciona en un laboratorio

La máquina virtual llega al archivo de paquetes de Ubuntu, PyPI, npm y GitHub, y a nada más, así que un laboratorio no puede apuntarse a un host de internet, y el servidor DNS del laboratorio no tiene adónde reenviar un nombre. Es una sola máquina virtual: el servidor y los clientes son espacios de nombres dentro de ella, así que los clientes no son máquinas separadas, y no hay un segundo servidor DHCP con el que competir.

## Acerca de

Hecho por [Mitsuru Technologies](https://mitsurutech.co.uk) para la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning): Python, Java, Go, Node, SQL y Linux en el navegador para escuelas y centros, con laboratorios de contenedores, laboratorios de máquina virtual, aplicaciones web desplegadas desde GitHub y cuestionarios que se califican solos. Prueba código sin cuenta en el [entorno de pruebas](https://mitsurutech.co.uk/es/playground).

Más laboratorios del mismo sitio: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) y [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licencia

Esta obra está bajo licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Puedes compartirla y adaptarla para aprender y enseñar, si citas a Mitsuru Technologies Ltd, no la usas comercialmente y compartes lo que hagas con ella bajo la misma licencia. El texto completo está en el archivo `LICENSE`.

Las aplicaciones que instala este laboratorio conservan las licencias de sus propios proyectos: [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) tiene licencia GPL-2.0 o GPL-3.0 y el cliente DHCP de ISC tiene licencia MPL-2.0; ambos se instalan desde el archivo de Ubuntu cuando arranca una máquina virtual, y nada de ellos se distribuye aquí.
