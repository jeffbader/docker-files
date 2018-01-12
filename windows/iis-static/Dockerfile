# escape=` 

# This is a docker example of a simple static website
# being served by IIS.
#
# NOTE:
# With the current release of the iis base image, you can't 
# use http://localhost to browse your site from the container host. 
# This is because of a known behavior in WinNAT, and will be 
# resolved in future. Until that is addressed, you need to use 
# the IP address of the container.
# REF:
#   Docker Hub, microsoft/iis
#   https://hub.docker.com/r/microsoft/iis/

# To build: docker image build -t imageTag .
# To run: docker container run -d imageTag
# To get container IP address: docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" containerId
# To access the website: open http://{containerIpAddress}

# FROM microsoft/iis 
FROM microsoft/windowsservercore
RUN powershell -Command Add-WindowsFeature Web-Server

SHELL ["powershell"]
ARG ENV_NAME=DEV
EXPOSE 80

# Configure IIS to log all entries to a file in C:\iislog, and to set the 
# maximum file size for log rotation—letting the log file grow to 4 GB. 
# REF: Docker on Windows, 
#      Elton Stoneman, 
#      Chapter 3: Developing Dockerized .NET and .NET Core Applications
#      Hosting IIS Applications in Docker
RUN Set-WebConfigurationProperty -p 'MACHINE/WEBROOT/APPHOST' `
    -fi 'system.applicationHost/log' -n 'centralLogFileMode' -v 'CentralW3C'; `    
    Set-WebConfigurationProperty -p 'MACHINE/WEBROOT/APPHOST' `
    -fi 'system.applicationHost/log/centralW3CLogFile' -n 'truncateSize' -v 4294967295; `    
    Set-WebConfigurationProperty -p 'MACHINE/WEBROOT/APPHOST' `
    -fi 'system.applicationHost/log/centralW3CLogFile' -n 'period' -v 'MaxSize'; `
    Set-WebConfigurationProperty -p 'MACHINE/WEBROOT/APPHOST' `
    -fi 'system.applicationHost/log/centralW3CLogFile' -n 'directory' -v 'C:/iislog'

# this example generates the index.html file 
# from a template file before putting index.html
# in the IIS wwwRoot directory for the default
# website
COPY wwwRoot/template.html C:/template.html

RUN (Get-Content -Raw -Path C:/template.html) ` 
    -replace '{hostname}', [Environment]::MachineName ` 
    -replace '{environment}', [Environment]::GetEnvironmentVariable('ENV_NAME') `
    | Set-Content -Path C:\inetpub\wwwroot\index.html 
 
 # Start IIS and monitor the IIS log file, tailing it to the console so IIS logs
 # are available via docker.  This command...
 #   - Starts the IIS Windows service (W3SVC) 
 #   - Makes an HTTP GET request to the localhost, which starts the IIS worker 
 #     process and writes the first log entry 
 #   - Flush the HTTP log buffer, so the log file gets written to the disk and 
 #     exists for PowerShell to watch 
 #   - Read the content of the log file in the tail mode, so any new lines written 
 #     to the file get shown on the console. 

# Hmm, this needs some work.  The base image is running ServiceMonitor via an Entryppoint,
# How do we override the base entrypoint, and run our tail task, and the service monitor?

# ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
# ENTRYPOINT []
CMD "Start-Service W3SVC; `
    Invoke-WebRequest http://localhost -UseBasicParsing | Out-Null; `
    netsh http flush logbuffer | Out-Null; "
#    Get-Content -path 'c:/iislog/W3SVC/u_extend1.log' -Tail 1 -Wait"
