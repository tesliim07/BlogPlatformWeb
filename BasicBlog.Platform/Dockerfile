# 1) Build & publish in the SDK image
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy csproj and restore dependencies
COPY BasicBlog.Platform.csproj ./
RUN dotnet restore BasicBlog.Platform.csproj

# Copy the rest of the source (Controllers/, Migrations/, etc.) and publish
COPY . ./
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# 2) Final image (still SDK so EF CLI is available)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
WORKDIR /app

# Install EF Core CLI globally & symlink it into the PATH
RUN dotnet tool install --global dotnet-ef --version 8.* \
 && ln -s /root/.dotnet/tools/dotnet-ef /usr/local/bin/dotnet-ef

# Copy the published output
COPY --from=builder /app/publish ./

# Copy project file & Migrations so dotnet-ef can locate your model
COPY BasicBlog.Platform.csproj ./
COPY Migrations ./Migrations

# Tell Kestrel to listen on 8080 and expose that port
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080

# On container start: apply migrations, then run your app
ENTRYPOINT ["bash","-lc","dotnet-ef database update --project BasicBlog.Platform.csproj && exec dotnet BasicBlog.Platform.dll"]
