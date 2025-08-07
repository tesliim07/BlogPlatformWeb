# build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release

WORKDIR /src

# 1) Copy project file and restore dependencies
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# 2) Copy source and build
COPY . ./
RUN dotnet build BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/build

# publish stage with migrations
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# bring in source and build outputs
COPY --from=build /src /src

# Apply EF Core migrations (ensure AZURE_SQL_CONNECTION_STRING env var is set)
RUN dotnet ef database update --no-build --project BasicBlog.Platform.csproj

# Publish the application
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# final runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
EXPOSE 8080 8081

# Copy published output
COPY --from=publish /app/publish .

# Start the application
ENTRYPOINT ["dotnet", "BasicBlog.Platform.dll"]