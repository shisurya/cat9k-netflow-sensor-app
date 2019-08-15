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
docker run -d -it --rm networkstatic/nflow-generator -t <ip> -p <port>
```

## Configuring the Switch
[Application hosting](https://wiki.cisco.com/display/C3A/KR+Port+Trunk+and+VLAN+Support#KRPortTrunkandVLANSupport-2.3App-hosting) and [Flexible Netflow](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-5/configuration_guide/nmgmt/b_165_nmgmt_9300_cg/b_165_nmgmt_9300_cg_chapter_01000.html) need to be enabled and configured on the switch. In addition, the switch management interface also needs to be configured.

