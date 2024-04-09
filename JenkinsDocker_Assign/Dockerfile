#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["JenkinsDocker_Assign/JenkinsDocker_Assign.csproj", "JenkinsDocker_Assign/"]
RUN dotnet restore "./JenkinsDocker_Assign/JenkinsDocker_Assign.csproj"
COPY . .
WORKDIR "/src/JenkinsDocker_Assign"
RUN dotnet build "./JenkinsDocker_Assign.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./JenkinsDocker_Assign.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "JenkinsDocker_Assign.dll"]

FROM jenkins/jenkins:lts
USER root
# Install dependencies and the .NET Core SDK
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        wget \
        gnupg \
        apt-transport-https \
        && \
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg && \
    mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/ && \
    wget -q https://packages.microsoft.com/config/debian/10/prod.list && \
    mv prod.list /etc/apt/sources.list.d/microsoft-prod.list && \
    chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg && \
    chown root:root /etc/apt/sources.list.d/microsoft-prod.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        dotnet-sdk-6.0
USER jenkins