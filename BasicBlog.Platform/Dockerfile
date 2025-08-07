# 1) Build & publish in the SDK image
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy csproj & restore
COPY BasicBlog.Platform/BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# Copy all source & publish
COPY BasicBlog.Platform/. ./
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# 2) Final image (with EF CLI)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install EF tool
RUN dotnet tool install --global dotnet-ef --version 8.* \
 && ln -s /root/.dotnet/tools/dotnet-ef /usr/local/bin/dotnet-ef

# Copy published output
COPY --from=builder /app/publish ./

# Copy csproj & Migrations for EF to see
COPY BasicBlog.Platform/BasicBlog.Platform.csproj ./
COPY BasicBlog.Platform/Migrations ./Migrations

ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080

ENTRYPOINT ["bash","-lc","dotnet-ef database update --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]
