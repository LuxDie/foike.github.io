---
layout:   blog
title:    Taller de Twister en Raspberry Pi
category: taller
tag:      maxigas taller workshop garagelab twister RaspberryPi twitter
ftr:	  ftr/0002-simondon.html
---

[Read it in English](/workshop/twister-pi.en.html)

__Etiquetas__: {{page.tags}}

![Twister on Raspberry Pi](/img/2014-09/twister-pi-leds.jpg)

El otro día fuimos al [Garage Lab](http://garagelab.cc) con
[maxigas](/con/maxigas.html) para instalar
[Twister](http://twister.net.co/) en una [Raspberry
Pi](http://raspberrypi.org/).  ¿Porqué una _Pi_?  Pues, porque maxigas llevó algunas...

# ¿Porqué Twister?

Twister brinda una alternativa a Twitter, distribuida. Twitter inventó
un excelente servicio para compartir rápidos cúmulos de información y
sentir la vibración de Internet.

Pero el servicio proprietario de Twitter viene de una sola empresa
cuyo negocio es analizar sus palabras y diseccionar su comportamiento
para beneficio de sus anunciantes. Esta empresa lee en sus tan
llamados "directos mensajes" (que se suponen privados), y puede
entregarlos a autoridades que no tienen jurisdicción legal sobre
ustedes (gracias a abusivas leyes anti-terroristas, y los "pedidos
discrecionales o sin permiso de divulgación" en la jurisdicción de
Twitter).

Puede ser censurado por autoridades hostiles al pueblo. Incluso
Twitter continúa poniendo barreras contra la participación
descentralizada, trabajando contra el espíritu de Internet.  Para su
beneficio ellos tienen un buen registro contra accesos abusivos a
sus--o sea, a datos suyos.

[![Captura de pantalla de la página de Twister](/img/2014-09/twister-pi-twister.net.co.png)](http://twister.net.co/)

En contraste, Twister es Software Libre, concebido desde sus comienzos
como un servicio para el mejor interés de sus usuarios, no sólo de los
intereses mercantiles de sus fundadores. Los "mensajes directos" (o
privados) en Twister realmente suceden entre pares, y están cifrados,
como cualquier otro de los mensajes comunes: nadie que no sea los que
participan puede leer nada. Más aún, nadie conoce su dirección de IP,
ni si están conectados o no. Twister está construido sobre tecnologías
punto a punto muy bien conocidas: la cadena de bloques pública de
[Bitcoin](https://es.wikipedia.org/wiki/Bitcoin), y el sistema de
distribución de
[Bittorrent](https://es.wikipedia.org/wiki/Bittorrent), incluyendo una
tabla distribuida de hashs (DHT) y un rastreador.

Una combinación tan espectacular alienta a una gran participación, y
provee rápidas actualizaciones. Esto permite una comunidad creciente de
participantes para escalar la red al costo de una fracción infinitesimal
de lo que significa el funcionamiento de la infraestructura de la
maquinaria de vigilancia centralizada de Twitter.

# Pre-Requisitos

  ![Se nota la impaciencia de los participantes](/img/2014-09/twister-pi-cantwait.jpg)

  Se necesitan:

  - una computadora con un procesador ARM, como una Raspberry Pi, una
    Olimex A10, etc.
  - una tarjeta SD (de 8GB o 16GB, y de Clase 10)
  - electricidad y conección a Internet
  - una computadora portatil conectada
  - si ya saben como hacerlo, una media hora; para desarrollar el
    taller hay que contar entre 2 y 4 horas, dependiendo de la
    experiencia de los participantes, y la riqueza de la explicación

# Primera etapa -- Instalar Raspbian

  ![Tarjetas SD de Clase 10](/img/2014-09/twister-pi-sdhc.jpg)

  1. Vaciar la tarjeta SD

     Este paso es opcional si la tarjeta es nueva. _Atención:_ la
     tarjeta puede aparecer como `/dev/sdb` u otro según el sistema
     operativo y los programas instalados.

         dd if=/dev/zero of=/dev/mmcblk0 bs=16M

  2. Descargar la imagen de Raspbian

     Guarda [el último torrent de
     Raspbian](http://downloads.raspberrypi.org/raspbian_latest.torrent)
     y descomprime la imagen en algún lugar.

  3. Graba la imagen de Raspbian

     Usa `dd` para grabar la imagen en la tarjeta SD. Asumiendo `/dev/mmcblk0`:

         dd if=2014-06-20-wheezy-raspbian.img of=/dev/mmcblk0 bs=16M 

     Después de menos de 10 minutos, la tarjeta debería estar lista
     para arrancar.


# Segunda etapa -- Arrancar la Pi con la tarjeta SD

  ![Una Pi antes de comersela](/img/2014-09/twister-pi-rpi.jpg)

  1. Insertar la tarjeta en la ficha
  2. Conectar el cable Ethernet entre la Pi y el router
  3. Enchufar la Pi (se puede usar también un puerto USB de la compu)

  Las LEDs de la Pi deberían titilar y después de unos segundos
  activarse.  Si esta etapa falla, consultar la sección _resolución de
  problemas_.


# Tercera étapa -- Identificar la Pi en la red

  ![Otra Pi todavía suelta](/img/2014-09/twister-pi-ddg.jpg)

  Existen varios métodos para descubrir la dirección IP asignada a la
  Pi por parte del servidor de DHCP.  Tenemos 3 documentados aquí.

## Por olfato

   Si la red es pequeña (por ejemplo, en casa), se puede deducir la
   dirección sabiendo la del laptop, y incrementando el último número
   (por el hecho de que el servidor DHCP usualmente ofrece las
   direcciones en orden creciente).  Es una buena idea cuando no hay
   muchas máquinas conectadas y quieran dar miedo a sus amigos
   ingenieros.

## arp-scan

   `arp-scan` es una herramienta escrita en Perl para analizar la red
   local y identificar las direcciones IP y MAC de las máquinas
   conectadas en su segmento de red.  Es simple y rápido:

   ![arp-scan -I wlan0 -local](/img/2014-09/twister-pi-arp-scan.jpg)

   La Raspberry Pi aparece como _\<Unknown\>_, porque la lista de
   direcciones MAC todavía no la reconoce.  Pero si estan por
   instalar algo, porque el programa no viene con su distro, ¿por qué
   no instalar `nmap`?

## nmap

   Desde hace años, `nmap` es la herramienta preferida tanto de los
   administradores de sistemas como de los hackers ofensivos.  Usando
   nmap es bastante fácil de indentificar la Raspberry Pi en su red
   local.  Se puede simplemente mapear la red entera y `nmap` va a
   mencionar la _Raspberry Foundation_ como fabricante del aparato
   que estan buscando.  Teoricamente, `arp-scan` podría hacerlo
   también, y va a hacerlo en el futuro, pero `nmap` mantiene más
   actualizada la lista de fabricantes.

   Si la red local es `192.0.2.0/24`:

       nmap -A 192.0.2.0/24 

   va a dar la lista de las máquinas activas y sus puertos
   interesantes, encima de su dirección IP, dirección MAC, y
   fabricante.  Una vez identificada la Pi, se pueden conectar a ella.

# Cuarta étapa -- Instalar los pre-requisitos

  ![Una Pi lista para comer](/img/2014-09/twister-pi-chair.jpg)

  La primera vez que se conectan a la Pi va a indicarles correr el
  programa `raspi-config`, que va a ofrecerles cambiar algunas
  opciones del sistema.  Si no van a usar la Pi para que trabaje con
  video pueden reducir la cantidad de memoria usada por el procesador
  gráfico en las opciones avanzadas.  Para el objetivo de este taller
  vamos a ignorar la configuración y seguir con la instalación.  Una
  vez entendido ¡no se olviden de mejorar la seguridad del aparato!

  Se conectan a la Pi a través de SSH.  Las credenciales por defecto
  son el usuario `pi`, con la contraseña `raspberry`.

      ssh pi@192.0.2.9
      sudo aptitude update
      sudo aptitude install git autoconf libtool build-essential libboost-all-dev libssl-dev libdb++ libminiupnpc-dev

   Nota: para más detallas, referirse a [la documentación oficial de
   Twister](https://github.com/miguelfreitas/twister-core/blob/master/doc/building-on-ubuntu-debian.md). La
   version de `db` debe ser 4.8 o 5.3, pero no 5.1 (por razón de
   incompatibilidad y instabilidad).

# Quinta étapa -- Compilar Twister

  Vamos a crear un usuario `twister`.

      adduser twister
      su - twister
      mkdir -m 0700 .twister
      git clone https://github.com/miguelfreitas/twister-html ~/.twister/html
      git clone https://github.com/miguelfreitas/twister-core
      cd twister-core
     ./bootstrap.sh && ./configure --disable-sse2 && make V=1

   Teclan control y la llave 'd' para salir del usuario 'twister' y
   volver a la cuenta de __root__, para correr `make install` desde la
   carpeta `/home/twister/twister-core`.

   ¡Nada más! Ahora Twister puede correr en la Pi. :)

  ![Tess tiene listo su Twister](/img/2014-09/twister-pi-tess.jpg)

# Sexta étapa -- Correr Twister

   Twister se usa por el comando `twisterd`:

       /usr/local/bin/twisterd -rpcuser=user -rpcpassword=pwd -rpcallowip=127.0.0.1

   Por ahora, las opciones de procesos remotos (RPC) estan escritas
   tal cual en el programa.  Se puede usar el archivo de configuración
   `$HOME/.twister/twister.conf` en vez de tipearlas cada vez en la
   linea de comando.  Sino se puede usar un script como lo siguiente:

## Un script para twister

        #!/bin/sh
        
        username="user"
        password="pwd"
        ipv4addr="127.0.0.1"
        
        TWISTER="/usr/local/bin/twisterd"
        ARGS="-rpcuser=$username -rpcpassword=$password -rpcallowip=$ipv4addr"
        
        exec $TWISTER $ARGS $@

   Guarden esto arriba en `/usr/local/bin/twister` o en otro lugar
   adentro del `PATH`.  Ahora usaremos `twister` para lanzar el
   programa.

## RPC inseguro sobre SSH

   Tenemos un problema: la interfaz web corre en localhost, pero nos
   conectamos con SSH.  Y no tenemos una pantalla a mano.  Sería
   bastante costoso en recursos de la Pi correr un navegador desde
   allá, y muy inseguro autorizar RPC remoto (¡cuán paradójico suena
   esto!)

   ¡Por suerte existen tuneles SSH! Pasando el puerto de la interfaz
   local de Twister en la computadora de control, podemos acceder al
   Twister "localmente", y además con cifrado. Para lograr una
   conección transparente al Twister desde nuestra máquina propia,
   vamos a configurar SSH adecuadamente.

        ### ~/.ssh/config
        ### raspberrypi
        ###
        Host pi
             HostName			192.0.2.123
             Port			22
             CheckHostIP		yes
             ForwardX11			no
             User			pi
             IdentityFile		~/.ssh/id_rsa
             RSAAuthentication		no
             PasswordAuthentication	no
             LogLevel			INFO
        # Aquí estamos: enviamos el puerto 28332 de la Pi a nuestro localhost
        # Sino, podemos correr: `ssh -Nf -L 28332:localhost:28332 pi` para abrir el tunel
        #     LocalForward		28332	127.0.0.1:28332


   Remplazar la dirección IP del `HostName` con la dirección real de
   la Pi.  Sería mejor asignar una dirección fija al Twister, pero si
   no hay muchos cambios en la red, la Pi debería quedarse con la
   misma dirección IP indefinidamente.  Antes de poder usar esta nueva
   configuración de SSH, conviene enviar su llave SSH en la Pi:

        ssh-copy-id -i ~/.ssh/id_rsa pi@192.0.2.123

   Listo.  Ahora, pueden usar Twister:

        ssh pi
        ssh -Nf -L 28332:localhost:28332 pi
        xdg-open http://localhost:28332

# Septima etapa -- ¡Bailamos el twist!

  Ahora tienen:

  - una Raspberry Pi funcionando con un Raspbian actualizado
  - Twister corriendo en la Pi
  - una interfaz Web segura para su Twister

  Todavía deben:

  - actualizar la cadena de bloques (__blockchain__)
  - registrar su cuenta de Twister
  - familiarizarse con la interfaz
  - finalizar la instalación

  ![la pizarra con una explicación de la cadena de bloques](/img/2014-09/twister-pi-blockchain.jpg)

  Cuando `twisterd` corre por primera vez, se conecta a la tabla
  distribuida (DHT) y empeza a descargar bloques.  Hoy en día se
  necesitan 5 minutos para volver al último bloque de los 52000+ en la
  cadena de Twister.  Antes haber descargado la cadena entera, no se
  puede hacer nada realmente útil.  Una vez descargada, pueden
  registrar su identificador personal.

  Registrarse en un proceso en dos pasos.  La verificación de la
  disponibilidad del nombre es instantánea, ya que la cadena de
  bloques contiene la colección completa de los nombres usados en la
  red.  Cuando su nick es único, pueden "guardarlo".  En este punto,
  twister esta "crunching" números para insertar en el próximo bloque
  un pedazo con el nombre.  Una vez que este bloque esta compartido,
  la red establece un consenso sobre la validez de su existencia, y
  después el nombre esta disponible para su uso.  Deberían pasar
  algunos minutos (entre 1 y 30) para que la red acepte un nuevo
  nombre, inscrito en el bloque corespondiente.  Durante este tiempo
  pueden personalizar su perfil.  En un momento van a recibir
  confirmación de la aparición del nombre en la cadena por una
  notificación de @newusers mencionándolo.  ¡Ahora su cuenta esta
  disponible para usar!

  _Unense a nosotrxs: @efecto99, @eisos, @maxigas, @minitrue, @tess._

# Resolución de problemas

  Por favor [reportar errores, dificultades,
  sugerencias](mailto:colectivo@foike.org?Subject=Twister%20en%20Pi)
  para que mejoremos este documento.

## La Raspberry Pi no arranca

   - Revisar el cable eléctrico: puede que la falta de potencia
     eléctrica dé un error en el booteo.
   - Revisar el adaptador de la tarjeta SD: si usan una micro-SD, es
     posible que no esté perfectamente enchufado
   - Controlar la calidad de la tarjeta SD.  Algunas tarjetas andan
     peor que otras, o no andan del todo.  Yo usé una Kingston 16GB
     Clase 10.

## Twister no compila

   - Referirse a la [documentación de
     compilación](http://twister.net.co/?page_id=29)

   ![El producto final corriendo en casa](/img/2014-09/twister-pi-final.jpg)