FROM mcr.microsoft.com/dotnet/sdk:10.0-alpine AS build

ARG BUILD_CONFIGURATION=Release

WORKDIR /src

COPY nuget.config .
COPY Directory.Build.props .
COPY Directory.Packages.props .
COPY src/Gateway.API/Gateway.API.csproj src/Gateway.API/

RUN --mount=type=secret,id=github_token \
    GITHUB_TOKEN=$(cat /run/secrets/github_token) && \
    dotnet nuget remove source github-fcg --configfile nuget.config || true && \
    dotnet nuget add source "https://nuget.pkg.github.com/FIAP-Cloud-Gaming/index.json" \
      --name github-fcg \
      --username FIAP-Cloud-Gaming \
      --password "$GITHUB_TOKEN" \
      --store-password-in-clear-text \
      --configfile nuget.config

RUN dotnet restore src/Gateway.API/Gateway.API.csproj

COPY . .

RUN dotnet publish src/Gateway.API/Gateway.API.csproj \
    -c $BUILD_CONFIGURATION \
    --no-restore \
    -o /app/publish \
    /p:UseAppHost=false

# ------------------------------------------------------------------------------

FROM mcr.microsoft.com/dotnet/aspnet:10.0-alpine AS final

LABEL org.opencontainers.image.title="Gateway API" \
      org.opencontainers.image.description="API Gateway YARP com JWT e Rate Limiting - FIAP Cloud Gaming" \
      org.opencontainers.image.source="https://github.com/FIAP-Cloud-Gaming/Gateway" \
      org.opencontainers.image.authors="Alessandro (DEV)"

WORKDIR /app
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

USER $APP_UID

COPY --from=build /app/publish .

ENTRYPOINT ["dotnet", "Gateway.API.dll"]