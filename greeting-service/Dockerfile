#Depending on the operating system of the host machines(s) that will build or run the containers, the image specified in the FROM statement may need to be changed.
#For more information, please see http://aka.ms/containercompat 

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2-nanoserver-1709 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/core/sdk:2.2-nanoserver-1709 AS build
WORKDIR /src
COPY ["greeting-service/greeting-service.csproj", "greeting-service/"]
RUN dotnet restore "greeting-service/greeting-service.csproj"
COPY . .
WORKDIR "/src/greeting-service"
RUN dotnet build "greeting-service.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "greeting-service.csproj" -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "greeting-service.dll"]
