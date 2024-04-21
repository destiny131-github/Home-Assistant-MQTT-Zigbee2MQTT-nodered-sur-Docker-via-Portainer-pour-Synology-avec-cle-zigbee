
# Home Assistant + MQTT + Zigbee2MQTT + nodered sur Docker via Portainer pour Synology avec cle zigbee sonoff 

<div style="padding: 10px; margin-bottom: 20px; border: 1px solid #ccc; border-radius: 5px;">
    <h2>Description</h2>
    <p>Ce tutoriel fournit les instructions pour déployer Home Assistant, un broker MQTT, nodered et Zigbee2MQTT sur un Synology NAS en utilisant Docker via Portainer. Ce setup permet de faciliter l'intégration et la gestion des appareils Zigbee dans votre réseau domestique.</p>
</div>

## Prérequis

<ul style="list-style-type:square;">
    <li>Synology NAS avec Docker et Portainer installés.</li>
    <li>macvlan mis en place avec une plage incluant 74 à 77.</li>
    <li>Accès à l'interface de Portainer.</li>
    <li>suivre le tutorial suivant qui est tres bien fait @robertklep </li> <a href="github.com/robertklep/dsm7-usb-serial-drivers/tree/main">https://github.com/robertklep/dsm7-usb-serial-drivers/tree/main</a>
</ul>

## Installation

### Étape 1: Configuration de Portainer

<ol>
    <li>Connectez-vous à l'interface web de votre Portainer.</li>
    <li>allez dans stack/editor .</li>
</ol>

### Étape 2: Déploiement de Home Assistant + MQTT + Zigbee2MQTT + nodered

<pre style="background-color: #f6f8fa; padding: 16px; border-radius: 8px; border: 1px solid #ccc;">
<code style="color: #0366d6;">
version: '3.8'

networks:
  default:
    external:
      name: Macvlan40

services:
  homeassistant:
    container_name: home-assistant
    image: homeassistant/home-assistant:stable
    volumes:
      - ./docker/homeassistant/config:/config   #changer l'adresse de votre volume
      - ./docker/homeassistant/localtime:/etc/localtime:ro #changer l'adresse de votre volume
    environment:
      - TZ=Europe/Paris
    restart: unless-stopped
    ports:
      - 8123:8123
    networks:
      default:
        ipv4_address: 192.168.2.74 #à modifier en fonction du reseau Macvlan
        
  nodered:
    container_name: nodered
    image: nodered/node-red:latest
    user: "1024:100"
    ports:
      - 1880:1880 
    volumes:
      - ./docker/homeassistant/nodered/data:/data  #changer l'adresse de votre volume
    restart: unless-stopped
    networks:
      default:
        ipv4_address: 192.168.2.75 #à modifier en fonction du reseau Macvlan
        
  zigbee2mqtt:
    image: koenkk/zigbee2mqtt
    container_name: zigbee2mqtt
    depends_on:
      - mosquitto
    volumes:
      - ./docker/homeassistant/zigbee2mqtt:/app/data  #changer l'adresse de votre volume
      - /run/udev:/run/udev:ro
    ports:
      - 8080:8080
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    restart: unless-stopped
    networks:
      default:
        ipv4_address: 192.168.2.76 #à modifier en fonction du reseau Macvlan

  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto:latest
    ports:
      - 1883:1883
      - 9001:9001
    volumes:
      - ./docker/mosquitto/data:/mosquitto/data      #changer l'adresse de votre volume
      - ./docker/mosquitto/log:/mosquitto/log        #changer l'adresse de votre volume
      - ./docker/mosquitto/config:/mosquitto/config  #changer l'adresse de votre volume
    restart: unless-stopped
    networks:
      default:
        ipv4_address: 192.168.2.77 #à modifier en fonction du reseau Macvlan
</code>
</pre>

### Étape 3: modification du fichier configuration.yaml de zigbee2mqtt

<li>il faut modifier le fichier comme ceci .</li>
<pre style="background-color: #f6f8fa; padding: 16px; border-radius: 8px; border: 1px solid #ccc;">
<code style="color: #0366d6;">
homeassistant: true
permit_join: true
frontend: 
  port: 8080
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://192.168.2.77:1883
serial:
  port: /dev/ttyACM0
  adapter: ezsp
advanced:
  network_key: [66, 44, 78, 220, 69, 99, 240, 153, 236, 96, 128, 253, 179, 158, 76, 102]
  pan_id: 33006
  ext_pan_id: [238, 229, 113, 212, 16, 93, 199, 50]
</code>
</pre>


### Étape 4: modification du fichier mosquitto.conf de mosquitto
<pre style="background-color: #f6f8fa; padding: 16px; border-radius: 8px; border: 1px solid #ccc;">
<code style="color: #0366d6;">
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
log_type all
listener 1883
allow_anonymous true
</code>
</pre>


### Utilisation

<li>redemarer les conteneurs .</li>
<p>accédez à vos conteneur via <a href="http://<adresse_ip_du_conteneur>+port">adresse_ip_du_conteneur>+port</a> pour configurer vos appareils.</p>

