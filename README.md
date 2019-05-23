# Code Example

Shows one webapi service calling another webapi service, within windows containers using docker-compose.

# Prerequisites
- dot net core 2.2.5
- docker
- Visual Studio 16.1
- Docker Desktop (on Windows), run using `windows containers`

# To Run
- Press F5
- Navigate to : https://localhost:44354/api/values

# Learnings

## dot net project creation
Dotnet can be used to create a `webapi` service using the command 

`dotnet new webapi`

## docker networking
Use the default bridge mode provided by docker. A service can be referenced using it's name, e.g. `http://greeting-service/` in `Startup.cs(32)`.


## docker config
There is a file called `docker-compose.override.yaml`, which contains the port mapping for the services.

