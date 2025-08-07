# 1) Build + restore
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy the csproj & restore dependencies
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# Copy all source files & compile
COPY . ./
RUN dotnet build BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/build

# 2) Publish stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Publish into a self-contained folder
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# 3) Runtime + migrations (using SDK so EF CLI is available)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install the EF Core CLI and symlink into PATH
RUN dotnet tool install --global dotnet-ef --version 8.* \
 && ln -s /root/.dotnet/tools/dotnet-ef /usr/local/bin/dotnet-ef

# Copy only your published binaries
COPY --from=publish /app/publish ./

# Copy the csproj & Migrations so the EF tool can discover your model
COPY BasicBlog.Platform.csproj ./
COPY Migrations ./Migrations

EXPOSE 8080

# At container start: run migrations, then launch your app
ENTRYPOINT ["bash","-lc","dotnet-ef database update --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]
