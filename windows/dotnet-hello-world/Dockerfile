# build stage
FROM microsoft/dotnet:1.1-sdk-nanoserver as build
WORKDIR /src 
COPY src/ .
RUN dotnet restore; dotnet publish

# final 
FROM microsoft/dotnet:1.1-runtime-nanoserver 
WORKDIR /dotnetapp 
COPY --from=build /src/bin/Debug/netcoreapp1.1/publish . 
CMD ["dotnet", "HelloWorld.NetCore.dll"] 
