# ================================
# Etapa 1 - Restauração de dependências
# (separada para aproveitar cache)
# ================================
FROM mcr.microsoft.com/dotnet/sdk:9.0-alpine AS base
WORKDIR /src
COPY *.csproj .
RUN --mount=type=cache,id=nuget-packages,target=/root/.nuget/packages \
    dotnet restore
 
# ================================
# Etapa 2 - Build e publicação
# ================================
FROM base AS build-production
ARG configuration=Release
COPY . .
RUN dotnet publish -c $configuration -o /app/publish /p:UseAppHost=false

# ================================
# Etapa 3 - Imagem final de produção
# ================================
FROM mcr.microsoft.com/dotnet/aspnet:9.0-alpine AS prod
ENV ASPNETCORE_URLS=http://*:5000
USER app
WORKDIR /app
COPY --from=build-production --chown=app:app /app/publish .
EXPOSE 5000
ENTRYPOINT ["dotnet", "ProjectOne.dll"]

# ================================
# Etapa 4 - Ambiente de desenvolvimento (com hot reload)
# ================================
FROM base AS dev
ENV DOTNET_USE_POLLING_FILE_WATCHER=true
WORKDIR /src
RUN curl -sSL https://aka.ms/getvsdbgsh | /bin/sh /dev/stdin -v latest -l /remote_debugger
COPY . .
EXPOSE 5000
ENTRYPOINT ["dotnet", "watch", "run", "--", "--urls", "http://0.0.0.0:5000"]
