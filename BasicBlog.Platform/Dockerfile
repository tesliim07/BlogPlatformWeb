# 1) Build + Publish (SDK image)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy project + restore + build + publish
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

COPY . ./
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# 2) Runtime + Migrations (SDK image again for EF CLI)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install EF Core CLI and symlink it into a PATH directory
RUN dotnet tool install --global dotnet-ef --version 8.* \
 && ln -s /root/.dotnet/tools/dotnet-ef /usr/local/bin/dotnet-ef

# Copy the published output
COPY --from=publish /app/publish .

# Copy the csproj + Migrations so dotnet-ef can find your model
COPY BasicBlog.Platform/BasicBlog.Platform.csproj ./
COPY BasicBlog.Platform/Migrations ./Migrations


EXPOSE 8080 8081

# On container start: run migrations (picks up AZURE_SQL_CONNECTION_STRING env-var)
# then start the app
ENTRYPOINT ["bash","-lc","dotnet-ef database update --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]

