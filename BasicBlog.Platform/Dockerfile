# publish stage (still running on the SDK image)
FROM build AS publish
WORKDIR /src

# apply EF Core migrations here (env-var for prod connection string must be available in build)
RUN dotnet ef database update --no-build --project BasicBlog.Platform.csproj

# then publish
RUN dotnet publish BasicBlog.Platform.csproj \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false

# final stage: runtime-only image, no CLI needed
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
EXPOSE 8080 8081

COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "BasicBlog.Platform.dll"]

