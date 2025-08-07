# 1) Build + Restore
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy csproj and restore
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# Copy everything and do a build
COPY . ./
RUN dotnet build BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/build

# 2) Migration + Publish
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# At this point /src still has your .csproj, obj/, and Migrations/
# Apply EF migrations (ensure your AZURE_SQL_CONNECTION_STRING env var is set)
RUN dotnet tool install --global dotnet-ef --version 8.* \
 && export PATH="$PATH:/root/.dotnet/tools" \
 && dotnet ef database update --no-build --project BasicBlog.Platform.csproj

# Now publish to a clean folder
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# 3) Runtime-only image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
EXPOSE 8080 8081

# Copy over only the published output
COPY --from=publish /app/publish .

# Start the app (no EF step here)
ENTRYPOINT ["dotnet", "BasicBlog.Platform.dll"]
