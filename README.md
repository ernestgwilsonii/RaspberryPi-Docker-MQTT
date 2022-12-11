# RaspberryPi-Docker-MQTT
Raspberry Pi Docker MQTT

```
##########################################
# Mosquitto MQTT Docker for Raspberry Pi #
#             REF: https://mosquitto.org #
##########################################


###############################################################################
ssh pi@IpAddressOfYourRaspberryPi
# Specify Mosquito Version:
# NORMAL VERSIONS: https://hub.docker.com/_/eclipse-mosquitto
# RASPBERRY PI VERSIONS: https://hub.docker.com/r/arm64v8/eclipse-mosquitto/
# The "openssl" version uses openssl instead of libressl, and enables TLS-PSK and TLS v1.3 cipher support
# REF: https://github.com/eclipse/mosquitto/tree/1c79920d78321c69add9d6d6f879dd73387bc25e/docker/2.0-openssl

# Set version variable:
MOSQUITTO_VERSION=2.0.15-openssl

# Download the Raspberry Pi version:
docker pull eclipse-mosquitto:$MOSQUITTO_VERSION

# List images and examine sizes
docker images

# Test running Mosquitto MQTT on Raspberry Pi
docker run -it eclipse-mosquitto:$MOSQUITTO_VERSION
# From another ssh session:
docker ps
###############################################################################


###############################################################################
# First time setup #
####################
ssh pi@IpAddressOfYourRaspberryPi

# Clone the kit onto the Raspberry Pi
cd /opt/docker-compose/W3GUY
git clone git@github.com:ernestgwilsonii/RaspberryPi-Docker-MQTT.git
cd RaspberryPi-Docker-MQTT

# Create bind mounted directories and copy starting files into place
mkdir -p /opt/docker/mqtt
mkdir -p /opt/docker/mqtt/config
mkdir -p /opt/docker/mqtt/config/conf.d
mkdir -p /opt/docker/mqtt/config/certs
mkdir -p /opt/docker/mqtt/data
mkdir -p /opt/docker/mqtt/log
chmod -R a+rw /opt/docker/mqtt
cp mosquitto.conf /opt/docker/mqtt/config/mosquitto.conf
cp passwd /opt/docker/mqtt/config/passwd
cp aclfile /opt/docker/mqtt/config/aclfile
cp TCP_1883_Unencrypted_MQTT.conf /opt/docker/mqtt/config/conf.d/TCP_1883_Unencrypted_MQTT.conf
cp TCP_8883_Encrypted_MQTT.conf /opt/docker/mqtt/config/conf.d/TCP_8883_Encrypted_MQTT.conf
cp TCP_9001_Unencrypted_Websockets.conf /opt/docker/mqtt/config/conf.d/TCP_9001_Unencrypted_Websockets.conf
cp TCP_9883_Encrypted_Websockets.conf /opt/docker/mqtt/config/conf.d/TCP_9883_Encrypted_Websockets.conf
cp generate-CA.sh /opt/docker/mqtt/config/certs/generate-CA.sh
cp passwd /opt/docker/mqtt/config/passwd
cp aclfile /opt/docker/mqtt/config/aclfile
chmod +x /opt/docker/mqtt/config/certs/generate-CA.sh
# Generate CA
bash -c "cd /opt/docker/mqtt/config/certs; /opt/docker/mqtt/config/certs/generate-CA.sh"
ls -alhF /opt/docker/mqtt/config/certs
# Generate certs for various other users/services
#bash -c "cd /opt/docker/mqtt/config/certs; /opt/docker/mqtt/config/certs/generate-CA.sh SomeUserName"
# Move hostname specific files to generic mqtt namming
mv /opt/docker/mqtt/config/certs/$(hostname).crt /opt/docker/mqtt/config/certs/mqtt.crt
mv /opt/docker/mqtt/config/certs/$(hostname).csr /opt/docker/mqtt/config/certs/mqtt.csr
mv /opt/docker/mqtt/config/certs/$(hostname).key /opt/docker/mqtt/config/certs/mqtt.key
# Reverse link naming so the generate-CA.sh script does not attempt to create these again
ln -P /opt/docker/mqtt/config/certs/mqtt.crt /opt/docker/mqtt/config/certs/$(hostname).crt
ln -P /opt/docker/mqtt/config/certs/mqtt.csr /opt/docker/mqtt/config/certs/$(hostname).csr
ln -P /opt/docker/mqtt/config/certs/mqtt.key /opt/docker/mqtt/config/certs/$(hostname).key
ls -alhF /opt/docker/mqtt/config/certs
# Set perms for bind mount as root and container group 1883
#chown -R root:1883 /opt/docker/mqtt


##########
# Deploy #
##########
# Deploy the stack into a Docker Swarm
docker stack deploy -c docker-compose.yml mqtt-stack
# docker stack rm mqtt-stack

# Verify
docker stack ls
docker service ls | grep mqtt-stack_MQTT
docker service logs -f mqtt-stack_MQTT


###################
# MQTT Basic Test #
###################
# Install MQTT client
# REF: https://mosquitto.org/download/

# Raspberry Pi client install:
apt install -y mosquitto-clients

# CentOS 7x Client
vi /etc/yum.repos.d/mqtt.repo
[home_oojah_mqtt]
name=mqtt (CentOS_CentOS-7)
type=rpm-md
baseurl=http://download.opensuse.org/repositories/home:/oojah:/mqtt/CentOS_CentOS-7/
gpgcheck=1
gpgkey=http://download.opensuse.org/repositories/home:/oojah:/mqtt/CentOS_CentOS-7/repodata/repomd.xml.key
enabled=1
:wq!
yum -y install mosquitto-clients

# Mac OS Client
brew install mosquitto

# Ubuntu Client
apt-get install mosquitto-clients

# Windows Client - https://mosquitto.org/download/

# Pub / Sub via command line (MQTT protocol)
# Subscribe to "all" channels/topics aka wildcard
#mosquitto_sub -v -h 127.0.0.1 -p 1883 -t "#"
# Subscribe to everything on Chan19
mosquitto_sub -v -h 127.0.0.1 -p 1883 -t "Chan19"
# Publish to Chan19
mosquitto_pub -d -h 127.0.0.1 -p 1883 -t "Chan19" -m "Hello MQTT from command line"

# Pub / Sub via browser (Websocket protocol)
# REF: http://mitsuruog.github.io/what-mqtt/
ws://IP:9001/mqtt
Chan19
Hello from Websocket!

# Next: https://github.com/mqttjs/MQTT.js/
###############################################################################


###############################################################################
# Generate Certs
# REF: https://mosquitto.org/documentation/
cd /opt/docker/mqtt/config/certs
./generate-CA.sh
# Generate certs for users/services
#./generate-CA.sh SomeUserName
#./generate-CA.sh SomeOtherUserName
#./generate-CA.sh SomeServiceName

# Add users
# REF: https://mosquitto.org/man/mosquitto_passwd-1.html
docker ps
docker exec -it ContainerIdHere sh
mosquitto_passwd -c /mosquitto/config/passwd SomeUserName
cat /mosquitto/config/passwd

# Edit the ACL controls
# REF: https://mosquitto.org/man/mosquitto-conf-5.html
# Either from inside the Docker container
vi /mosquitto/config/aclfile
# Or from the bind mounted Docker host
vi /opt/docker/mqtt/config/aclfile
###############################################################################
```
