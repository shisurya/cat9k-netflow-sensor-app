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
[Application hosting](https://wiki.cisco.com/display/C3A/KR+Port+Trunk+and+VLAN+Support#KRPortTrunkandVLANSupport-2.3App-hosting) and [Flexible Netflow](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-5/configuration_guide/nmgmt/b_165_nmgmt_9300_cg/b_165_nmgmt_9300_cg_chapter_01000.html) need to be enabled and configured on the switch. For application hosting to work, a Cisco USB SSD drive must be plugged into the back-panel USB port of the switch. In addition, the switch management interface also needs to be configured.

Configuring Application Hosting:

The details about how to configure application hosting on the switch can be found [here](https://wiki.cisco.com/display/C3A/KR+Port+Trunk+and+VLAN+Support#KRPortTrunkandVLANSupport-2.3App-hosting).

```
app-hosting appid nfsen_app
 app-vnic AppGigabitEthernet trunk
  guest-interface 0
 app-resource docker
```

Configuring Flexible Netflow:

The details about how to configure the switch to export netflow records can be found [here](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-5/configuration_guide/nmgmt/b_165_nmgmt_9300_cg/b_165_nmgmt_9300_cg_chapter_01000.html).

1. Creating a flow record:

The flow record is a description of the elements present in the Netflow template. It offers the administrator options on what they want to see in the flow data.

```
flow record record1
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
```

2. Creating a flow exporter:
 
The exporter defines how the flows process out of the device to the collector, and any options relative to that export. 

 ```bash
flow exporter export1
  destination <docker_app_ip>
  transport udp 2055
```

3. Creating a flow monitor:

The flow monitor ties together the flow record and the flow exporter. 

 ```bash
flow monitor monitor1
  exporter export1
  cache timeout inactive 350
  cache timeout active 350
  record record1
```

4. Applying the flow monitor to interfaces:

In this step, we get into interface configuration mode, and apply the flow monitor in input or output mode, or both.

```bash
interface GigabitEthernet1/0/1
 ip flow monitor monitor1 input
 
interface GigabitEthernet1/0/2
 ip flow monitor monitor1 output
```

Configuring the management interface (GigabitEthernet0/0):

```
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip address <mgmt_ip_address> <mgmt_ip_netmask>
 negotiation auto
```

## Deploying the application to the switch

1. Save the docker application image as a tarball:

```
docker save -o nfsen_app.tar nfsen-app
```

2. First copy the application tarball to the tftp server. Then, copy the application tarball to the USB SSD inserted into the switch:

```
switch#copy tftp://<tftp_server_ip_address>/nfsen_app.tar flash:
```

3. Install and activate the application on the switch. The 'activate' step allocates the necessary system resources to the application.

```
switch#app-hosting install appid nfsen_app package flash:nfsen_app.tar
Installing package 'flash:nfsen_app.tar' for 'nfsen_app'. Use 'show app-hosting list' for progress.

switch#show app-hosting list                                                      
App id                                   State
---------------------------------------------------------
nfsen_app                               DEPLOYED

switch#app-hosting activate appid nfsen_app
nfsen_app activated successfully
Current state is: ACTIVATED
```

4. Run the application on the switch:

```
switch#app-hosting start appid nfsen_app
nfsen_app started successfully
Current state is: RUNNING

switch#show app-hosting list                    
App id                                   State
---------------------------------------------------------
nfsen_app                                RUNNING
```

To access the output webpage from a web browser on the local machine, use ssh forwarding:

```bash
ssh -L 127.0.0.1:9999:<docker_app_ip>:80 username@server_ip
```
Point your web browser at http://localhost:9999 You will see the nfsen home page.

