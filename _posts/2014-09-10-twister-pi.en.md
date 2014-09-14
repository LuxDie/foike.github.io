---
layout:   blog
title:    Twister on Raspberry Pi Workshop
category: workshop
tag:      maxigas taller workshop garagelab twister RaspberryPi twitter
ftr:	  ftr/0002-simondon.html
---

[Leelo en Castellano](/taller/twister-pi.html)

__Tags__: {{page.tags}}

![Twister on Raspberry Pi](/img/2014-09/twister-pi-leds.jpg)

The other day we went to the [Garage Lab](http://garagelab.cc)
with [maxigas](/with/maxigas.html) to install
[Twister](http://twister.net.co/) on a [Raspberry
Pi](http://raspberrypi.org/).  Why a _Pi_?  Well, because maxigas
brought some...

# Why Twister?

  Twister provides a distributed alternative to Twitter.  Twitter
  invented a great service to share fast bits of information and feel
  the hum of the Internet.

  But Twitter's proprietary service comes from a single company whose
  business is to analyze your words and dissect your behavior for the
  benefit of advertisers.  It reads your so-called "direct messages",
  and may give them away to unwarranted authorities in your
  jurisdiction (thanks to abusive anti-terrorist laws, and "gag
  orders" in their's).  It may be censored by arbitrary authorities
  hostile to the people.  Moreover, Twitter keeps putting barriers to
  decentralized participation, working against the spirit of the
  Internet.  To their credit, they have a good track record against
  unwarranted access to their--er, your data.

  [![Screenshot of http://twister.net.co/](/img/2014-09/twister-pi-twister.net.co.png)](http://twister.net.co/)

  In contrast, Twister is free software, conceived from the beginning
  to serve the best interest of its users, not the sole mercantile
  interest of its founders.  "Direct messages" on Twister really occur
  between you and your peer, and are encrypted, like any regular
  message: non-participants cannot read anything.  Moreover, nobody
  knows your IP address, nor whether you're online or not.  Twister is
  built on well-known peer-to-peer technologies: the public ledger
  (_blockchain_) of [Bitcoin](https://en.wikipedia.org/wiki/Bitcoin),
  and the distribution system of
  [Bittorrent](https://en.wikipedia.org/wiki/Bittorrent), comprising a
  DHT and a tracker.  Such a great combination encourages wide
  participation, and provides fast updates.  It allows a growing
  community of participants to scale up the network at an
  infinitesimal fraction of the cost of running the centralized
  infrastructure of the Twitter surveillance machine.

# Pre-Requisites

  ![Here we can notice some impatience among the participants](/img/2014-09/twister-pi-cantwait.jpg)

  You will need:

  - a computer with an ARM processor, such as a Raspberry Pi, an
    Olimex A10, etc.
  - an SD card (of 8GB or 16GB, and preferrably Class 10)
  - electricity and an Internet connection
  - a connected laptop
  - about half an hour if you know what you're doing ; to develop the
    workshop, count between two to four hours, depending on the
    experience of the participants, and the richness of the
    explanation.

# First step -- Install Raspbian

  ![A bunch of Class 10 SD cards](/img/2014-09/twister-pi-sdhc.jpg)

  1. Zero-out the SD card

     This step is optional if the card is new. _Warning:_ the card may
     appear as `/dev/sdb` or something else according to your
     operating system and the installed programs. YMMV.

         dd if=/dev/zero of=/dev/mmcblk0 bs=16M

  2. Download the Raspbian image

     Save the [latest Raspbian
     torrent](http://downloads.raspberrypi.org/raspbian_latest.torrent)
     and unzip the image somewhere.

  3. Burn the Raspbian image

     Use `dd` to burn the image to the SD card.  Assuming `/dev/mmcblk0`:

         dd if=2014-06-20-wheezy-raspbian.img of=/dev/mmcblk0 bs=16M 

     After less than 10 minutes, the SD card should be ready to boot.

# Second step -- Boot the Pi from the SD card

  ![A Pi before being eaten](/img/2014-09/twister-pi-rpi.jpg)

  1. Insert the SD card in the slot
  2. Connect the Ethernet cable between the Pi and the router
  3. Power up the Pi (you may as well plug it into an USB port of your
     laptop)

  The Pi's LEDs should blink, then after a few seconds turn on.  If
  that step fails, please refer to the _Troubleshooting_ section.

# Third step -- Identify the Pi on the network

  ![Another lonely Pi](/img/2014-09/twister-pi-ddg.jpg)

  There are several methods to discover the IP address assigned by the
  DHCP server to the Pi.  Here are three of them.

## Wild Guess

   On a small network (e.g., at home), you can figure out the IP from
   your laptop's, and incrementing the last number (because the DHCP
   server usually assigns addresses in ascending order).  It's a great
   method when there are not many connected devices and you want to
   scare your engineer friends.

## arp-scan

   `arp-scan` is a network analysis tool written in Perl, to identify
   IP and MAC addresses of devices connected to the local network.
   It's fast and easy to use:

   ![arp-scan -I wlan0 -local](/img/2014-09/twister-pi-arp-scan.jpg)

   The Raspberry Pi appears as _\<Unknown\>_, because the manufacturer
   does not appear yet in the MAC list used by `arp-scan`.  Anyway, if
   you're going to install some program because it does not come with
   your distro, why not install `nmap`?

## nmap

   For years, `nmap` has been a favorite tool of sysadmins and
   offensive hackers alike.  Using nmap makes it easy to identify the
   Raspberry Pi on your local network.  Scanning the entire network
   will turn up the Pi: `nmap` will correctly identify the _Raspberry
   Foundation_ as the manufacturer of the device we're looking for.
   Theoretically, `arp-scan` could do it as well, and will certainly,
   but the `nmap` community maintains a fresher list of network device
   manufacturers.

   Considering a local IP range on `192.0.2.0/24`:

       nmap -A 192.0.2.0/24 

   will return a list of active hosts, with their interesting service
   ports, in addition to the IP address, the MAC address, and the
   manufacturer.  Once the Pi is identified, we can connect to it.

# Fourth step -- Install required packages

  ![A Pi ready to be eaten](/img/2014-09/twister-pi-chair.jpg)

  When you connect to the Pi for the first time, it will recommend
  running the `raspi-config` program that allows changing some options
  of the system.  If you're not planning on using the Pi for
  video-intensive work, you can reduce the amount of RAM shared with
  the graphical processor in the advanced options.  For the purpose of
  this workshop, we'll skip configuration and focus on the
  installation.  Once you have understood the process, don't forget to
  secure your Pi!

  You connect to the Pi via SSH.  The default credentials are the
  login `pi` with the password `raspberry`.

      ssh pi@192.0.2.9
      sudo aptitude update
      sudo aptitude install git autoconf libtool build-essential libboost-all-dev libssl-dev libdb++ libminiupnpc-dev

   Note: for more details, please refer to the [official Twister
   documentation](https://github.com/miguelfreitas/twister-core/blob/master/doc/building-on-ubuntu-debian.md).
   The version of `db` should be 4.8 or 5.3, but not 5.1 (for reasons
   of incompatibility and unstability).

# Fifth step -- Compile Twister

  Let's create a dedicated `twister` user.

      adduser twister
      su - twister
      mkdir -m 0700 .twister
      git clone https://github.com/miguelfreitas/twister-html ~/.twister/html
      git clone https://github.com/miguelfreitas/twister-core
      cd twister-core
     ./bootstrap.sh && ./configure --disable-sse2 && make V=1

  Hit Control and the 'd' key to exit from the 'twister' user and come
  back to the __root__ account, and run `make install` from the
  `/home/twister/twister-core` directory.

  That's it!  Now Twister can run on the Pi. :)

  ![Tess got her Twister ready](/img/2014-09/twister-pi-tess.jpg)

# Sixth step -- Run Twister

  To use Twister, run the `twisterd` command:

       /usr/local/bin/twisterd -rpcuser=user -rpcpassword=pwd -rpcallowip=127.0.0.1

  In the current alpha version, the remote procedure calls (RPC) options are hardwritten in the program.  The configuration file in `$HOME/.twister/twister.conf` can be used to store them instead of typing them each time on the command line.  Otherwise, you can use a script like the following one:

## A script to run Twister

        #!/bin/sh
        
        username="user"
        password="pwd"
        ipv4addr="127.0.0.1"
        
        TWISTER="/usr/local/bin/twisterd"
        ARGS="-rpcuser=$username -rpcpassword=$password -rpcallowip=$ipv4addr"
        
        exec $TWISTER $ARGS $@

   Save the executable to `/usr/local/bin/twister` or another place in
   your `PATH`.  From now on we can use `twister` to run the program.

## Insecure RPC over SSH

   We have a problem: the web interface runs on localhost, but we're
   connecting via SSH.  We don't have a display attached either.  It
   would be quite heavy on the Pi to run a web browser directly from
   there, and very insecure to authorize remote RPC (how paradoxical
   does that sound!)

   Fortunately, we can use an SSH tunnel!  By passing the port of
   Twister's local Web interface to the controlling laptop, we can
   access to Twister "locally", and even better: through an encrypted
   channel.  In order to setup a transparent connection from our own
   laptop, let's setup a proper SSH configuration.

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
        # Here we are: sending the port 28332 from the Pi to our localhost
        # Otherwise, running: `ssh -Nf -L 28332:localhost:28332 pi` will open the tunnel
        #     LocalForward		28332	127.0.0.1:28332

   Replace the IP address on the `HostName` line with the actual IP
   address of the Pi.  It would be better to assign a fixed IP address
   to Twister, but if the network topology doesn't change much, the Pi
   should keep its address indefinitely.  Before we can use that new
   SSH configuration, we must send our SSH key to the Pi:

        ssh-copy-id -i ~/.ssh/id_rsa pi@192.0.2.123

   Done.  Now we can launch Twister:

        ssh pi
        ssh -Nf -L 28332:localhost:28332 pi
        xdg-open http://localhost:28332

# Seventh step -- ¡Let's do the twist!

  Now you have:

  - a functional Raspberry Pi running an up-to-date Raspbian
  - Twister running on the Pi
  - a secure Web interface to your Twister

  You still need to:

  - update the blockchain
  - register your Twister account
  - get used to the user interface
  - finalize the installation

  ![the board with an explanation of the blockchain](/img/2014-09/twister-pi-blockchain.jpg)

  When `twisterd` runs for the first time, it connects to the
  Distributed Hash Table (DHT) and starts downloading the blocks.
  Today about 5 minutes are needed to catch up with the 52000+ blocks
  that form the Twister blockchain.  Before you have downloaded the
  entire blockchain, there's nothing much useful to do with Twister.
  Once it's complete, you can register your account.

  Registration is a two-step process.  Verifying the availability of
  your chosen name is instantaneous, as the blockchain contains the
  whole collection of registered names on the network.  Once you chose
  a unique name, you can "save it".  At this point, Twister is
  crunching numbers to insert into the next block a chunk with your
  chosen name.  After this block is shared, the network establishes a
  consensus on the validity of its existence, and then the name
  becomes available for use.  It should take a few minutes, between
  one and thirty, for the network to accept the corresponding block.
  Meanwhile, you can use this time to customize your profile.
  Shortly, you're going to receive a confirmation that the name
  appeared in the blockchain in the form of a notification from
  @newusers mentioning your chosen name.  Finally, your Twister
  account is ready to use!

  _Follow us on Twister: @efecto99, @eisos, @maxigas, @minitrue,
  @tess._

# Troubleshooting

  Please [report errors, difficulties,
  suggestions](mailto:colectivo@foike.org?Subject=Twister%20on%20Pi)
  so that we can better this document.

## The Raspberry Pi doesn't boot

   - Review the electrical cable: it's possible that an electrical
     unstability cause an error during the boot.
   - Check the SD card adaptor if you're using a micro-SD, as it could
     not be a perfect fit for the Pi's socket.
   - Control the quality of the SD card.  Some cards work better than
     others, or do not work at all.  I used a Kingston 16GB Class 10. 

## Twister compilation fails

   - Refer to the [compilation documentation](http://twister.net.co/?page_id=29)

   ![The final product running at home](/img/2014-09/twister-pi-final.jpg)