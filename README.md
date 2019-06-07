This repository contains two webapi services. One web services calls the other using http.

Using these two services as examples the following deployment sceanarios are illustrated:

- running the services as separate services inside visual studio
- running the services as separate Docker Windows containers inside visual studio
- running the services as as separate Docker Windows containers from the command line of a Dev machine
- running the services as a Docker Compose set inside Visual Studio
- running the services as a Docker Compose set from the command line of a Dev machine
- running the services as a Docker Swarm set from the command line of a Windows Server 2019

Each scenario builds on each other so they should be run in order.

## Run as Separate Services inside visual studio

### Prerequisites
Have the following versions or later:
- Dev machine is Windows 10 Version 1806
- Visual Studio Version 16.1
- Dot Net Core 2.2.5
- Hyper-V enabled on Dev machine
- Docker Desktop, running with `windows containers`

### Instructions
- Open `greeting.sln` in Visual Studio
- Right Click `Solution greeting` in solution explorer > Set Startup projects
- Choose multiple startup projects and start `greeting-service` then `greeting-controller`
- Press `F5`
- Should see a browser open at `https://localhost:44344/api/values` displaying text `["[\"value1\",\"value2\"]"]`.

## Runs as Docker Compose in Visual Studio
- Open `greeting.sln` in Visual Studio
- Right Click `docker-compose` > `Set as StartUp Project` 
- Press `F5`
- Should see a browser open at `https://<ip-address>/api/values` displaying text `["[\"value1\",\"value2\"]"]`.

## Run as Separate Docker Containers on command line on Dev Machine

### Prerequisites
- Create an account on https://hub.docker.com/ and note the `dockerId`.
- The powershell 'curl' command (pre-v6) does not allow certificate errors to be ignored. To so do, run the following once per powershell session:
```powershell
add-type @"
    using System.Net;
    using System.Security.Cryptography.X509Certificates;
    public class TrustAllCertsPolicy : ICertificatePolicy {
        public bool CheckValidationResult(
            ServicePoint srvPoint, X509Certificate certificate,
            WebRequest request, int certificateProblem) {
            return true;
        }
    }
"@
[System.Net.ServicePointManager]::CertificatePolicy = New-Object TrustAllCertsPolicy
```

### Instructions

```powershell
cd <directory containing docker-compose.yml>
docker image build -t <dockerId>/greetingcontroller -f .\greeting-controller\Dockerfile .
docker image build -t <dockerId>/greetingservice -f .\greeting-service\Dockerfile .
docker run -d -p 8080:80 <dockerId>/greetingcontroller
docker run -d -p 8081:80 <dockerId>/greetingservice
curl http://localhost:8080/api/values
```

- Delete the containers:
    - `docker rm -f $(docker ps -aq)`

### Notes
- In the class `greeting_controller.Startup` the IP address of the Docker host is found by using the domain name `host.docker.internal`.


## Run as Docker Compose on command line on Dev Machine
- `cd <directory containing docker-compose.yml>`
- `docker-compose up -d`
- Get the IP of the container on the  NAT network created by docker-compose. Using localhost or 127.0.0.1 will not work.
- `$ip = docker container ls --format='{{json .}}' | convertfrom-json | ? {$_.Image -eq 'greetingcontroller'} | % {docker inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}'  $_.ID}`


- `write-host $ip`
- `curl https://$ip/api/values`
- Stop the services: `docker-compose stop`



## Run as Separate Docker Containers on command line on Windows Server

### Prerequisites

- On the *Dev machine* push the image to docker repository:

```powershell
# Login into docker hub
docker login -u <dockerId> -p <password>
docker push <dockerId>/greetingcontroller
docker push <dockerId>/greetingservice
```

- Use Windows Server 2019 Version 1803 or later. It is supposed to work on Windows Server 2016, but this has not been verified.
- If this Windows Server is itself a Virtual Machine, then hardware virtualization must be enabled. E.g. for Hyper-V, on the Hyper-V host, run the powershell command:
`Get-VM WinContainerHost | Set-VMProcessor -ExposeVirtualizationExtensions $true`
- Run the following powershell commands as administrator on the *Windows Server*:

```powershell
# Turn off firewall
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Uninstall-WindowsFeature Windows-Defender

# Turn on container support
Install-WindowsFeature -Name Containers
Restart-Computer -Force  

# Install Docker
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name docker -ProviderName DockerMsftProvider

# Test
## Login into dockerhub.com
docker login -u <dockerId> -p <password>
docker container run -it mcr.microsoft.com/windows/nanoserver:1809 ipconfig
 
```
- Reference: https://computingforgeeks.com/how-to-run-docker-containers-on-windows-server-2019/



## Instructions

```powershell
cd c:\users\<usrname>\Documents
mkdir greeting
cd greeting

# Pull images from docker hub
docker pull <dockerId>/greetingcontroller
docker pull <dockerId>/greetingservice
docker swarm init --advertise-addr 127.0.0.1

# Create compose file
$composefile = @"
version: '3.4'

services:
  greeting-controller:
    image: pngan/greetingcontroller:v1
    ports:
        - 8080:80

  greeting-service:
    image: pngan/greetingservice:v1
"@
$composefile | Out-File -FilePath docker-compose.yml -Encoding ascii -Force

docker stack deploy --compose-file .\docker-compose.yml greetingstack

ipconfig
# Note the IP Address of this Windows Server machine - we'll use it in the next step

````

- !!! IMPORTANT !!!  On machine that is NOT the Windows Server: On a Web browser and navigate to
http://windows-server-ip-addr:8080/api/values The reason for this is that the published port on windows containers cannot be reached from loopback addresses: https://blog.sixeyed.com/published-ports-on-windows-containers-dont-do-loopback/  And even though this article says that it is no longer a problem in Windows 1809 onwards, it was still encountered in Windows Server Version 1809.

# Code Notes

## dot net project creation
Dotnet can be used to create a `webapi` service using the command 

`dotnet new webapi`

## docker networking
Use the default bridge mode provided by docker. A service can be referenced using it's name, e.g. `http://greeting-service/` in `Startup.cs(32)`.

## docker config
There is a file called `docker-compose.override.yaml`, which contains the port mapping for the services.

