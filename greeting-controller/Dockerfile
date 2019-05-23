#Depending on the operating system of the host machines(s) that will build or run the containers, the image specified in the FROM statement may need to be changed.
#For more information, please see http://aka.ms/containercompat 

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2-nanoserver-1709 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/core/sdk:2.2-nanoserver-1709 AS build
WORKDIR /src
COPY ["greeting-controller/greeting-controller.csproj", "greeting-controller/"]
RUN dotnet restore "greeting-controller/greeting-controller.csproj"
COPY . .
WORKDIR "/src/greeting-controller"
RUN dotnet build "greeting-controller.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "greeting-controller.csproj" -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "greeting-controller.dll"]
