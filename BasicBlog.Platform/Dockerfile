```dockerfile
# build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy project and restore
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# Copy source and build
COPY . ./
RUN dotnet build BasicBlog.Platform.csproj -c $BUILD_CONFIGURATION -o /app/build

# publish stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy source and build outputs
COPY --from=build /src /src

# Publish app
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# final stage: SDK image for migrations and runtime
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install EF Core CLI as global tool
RUN dotnet tool install --global dotnet-ef --version 8.*

# Add the global tools path
ENV PATH="/root/.dotnet/tools:${PATH}"

# Copy published output
COPY --from=publish /app/publish ./

# Copy EF project & migrations so dotnet-ef can locate context
COPY BasicBlog.Platform.csproj ./
COPY Migrations ./Migrations

EXPOSE 8080 8081

# Run migrations using the global tool 'dotnet-ef', then start the app
ENTRYPOINT ["bash", "-lc", "dotnet-ef database update --no-build --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]
```
