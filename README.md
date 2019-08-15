# Cisco Catalyst 9K Netflow Sensor Application
A docker container based Netflow collector and web display application for Cisco Catalyst 9K switch. The application is built on [NFSEN](http://nfsen.sourceforge.net/) and [NFDUMP](http://nfdump.sourceforge.net/) which are documented and hosted at [SourceForge.net](https://sourceforge.net/)

## Building the Application
Prerequisites:
* Docker installed
* Internet connection

The application can be built using the [docker build](https://docs.docker.com/engine/reference/commandline/build/) command:

```bash
cd .\cat9k-netflow-sensor-app-master\
docker build -t nfsen-app .
```

Use [docker images](https://docs.docker.com/engine/reference/commandline/images/) command to check if the image is successfully installed:

```bash
docker images

REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
nfsen-app                       latest              982603daad1d        50 seconds ago      590MB
```

## Testing the Application
For testing, the application can be run locally on the same machine where it is built. Use [docker run](https://docs.docker.com/engine/reference/commandline/run/) command to run the container:

```bash
docker run -p 81:80 -p 2055:2055/udp -p 4739:4739/udp -p 6343:6343/udp -p 9996:9996/udp  -i -t --name nfsen_app_run nfsen-app
```

If the Docker startup is successful, it will print a large amount of debugging information, which culminates with:

```bash
INFO exited: nfsen_start (exit status 0; expected)
```

Point your web browser at http://localhost:81 You will see the nfsen home page.

To generate and export mock flow data to the nfsen application, run the netflow generator application:

```bash
docker run -d -it --rm networkstatic/nflow-generator -t <ip> -p 2055
```

## Configuring the Switch
[Application hosting](https://wiki.cisco.com/display/C3A/KR+Port+Trunk+and+VLAN+Support#KRPortTrunkandVLANSupport-2.3App-hosting) and [Flexible Netflow](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-5/configuration_guide/nmgmt/b_165_nmgmt_9300_cg/b_165_nmgmt_9300_cg_chapter_01000.html) need to be enabled and configured on the switch. In addition, the switch management interface also needs to be configured.

Configuring Flexible Netflow:

The details about how to configure the switch to export netflow records can be found [here](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-5/configuration_guide/nmgmt/b_165_nmgmt_9300_cg/b_165_nmgmt_9300_cg_chapter_01000.html). Following is an example:

1. Creating a flow record:

```
flow record record1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
```
2. Creating a flow exporter:
 
 ```bash
flow exporter export1
  destination <app_ip_address>
  transport udp 2055
```

3. Creating a flow monitor:

 ```bash
flow monitor monitor1
  exporter export1
  cache timeout inactive 350
  cache timeout active 350
  record record1
```

4. Applying the flow monitor to interfaces:

```bash
interface GigabitEthernet1/0/1
 ip flow monitor monitor1 input
 
interface GigabitEthernet1/0/2
 ip flow monitor monitor1 input
```
