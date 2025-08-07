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
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Publish app
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# final stage: SDK image for migrations
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install EF CLI
RUN dotnet tool install --global dotnet-ef --version 8.* \
    && export PATH="$PATH:/root/.dotnet/tools"
ENV PATH="$PATH:/root/.dotnet/tools"

# Copy published output
COPY --from=publish /app/publish ./

# Copy EF project files for migrations
COPY BasicBlog.Platform.csproj ./
COPY Migrations ./Migrations

EXPOSE 8080 8081

# Run migrations then start
ENTRYPOINT ["bash","-lc","dotnet ef database update --no-build --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]
