# escape= `

# It seems like you should be able to use the windows nanoserver
# for running kibana, but there are some issues...
# REF: search... kibana nanoserver
# REF: https://github.com/elastic/kibana/issues/11405
# For now, stick with servercore
#FROM openjdk:8-jdk-nanoserver
FROM openjdk:8-jdk-windowsservercore
SHELL ["powershell"]
# SHELL [ "powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Elasticsearch
ENV ES_VERSION="6.1.1" `
    KIBANA_HASH="1d9138e6d6700bf33f4a3931a630fa6d3aa8675a58850fe2ee9f168aa11a270335af1e17d83dcc7e8cf3dcda1d5376963dde743aca9031ea5035caa8c13e9832" `
    KIBANA_HOME="c:/kibana" `
    ELASTICSEARCH_URL="http://elasticsearch:9200"

RUN Invoke-WebRequest $('https://artifacts.elastic.co/downloads/kibana/kibana-{0}-windows-x86_64.zip' -f $env:ES_VERSION) -OutFile 'kibana.zip' -UseBasicParsing; `
    if ((Get-FileHash kibana.zip -Algorithm sha512).Hash -ne $env:KIBANA_HASH) { Write-Error 'Hash mismatch'; exit 1}; `
    Expand-Archive kibana.zip -DestinationPath c:\ ; `
    Move-Item $('c:/kibana-{0}-windows-x86_64/' -f $env:ES_VERSION) $env:KIBANA_HOME; `
    Remove-Item kibana.zip

# On windows, Volumes get mapped via symlinks to directories on the
# host.  Some languages have problems resolving those symlinks in Docker.
# So...we map the volume to a logical drive, and then configure the app
# to use the logical drives.
# REF: https://github.com/moby/moby/issues/27537

# For this image that means:
#    We want a volume for kibana data directory
#    In the container, we use c:\kibana-data as the Volume
#    We map c:\kibana-data to D:\
#    We configure kibana to write it's data to the D:

VOLUME ["c:/kibana-data"]

RUN Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name ServerPriorityTimeLimit -Value 0 -Type DWord; `
    Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices'  -Name 'D:' -Value '\??\C:\kibana-data' -Type String; 

# Default configuration for host & Elasticsearch URL
WORKDIR C:/kibana
RUN (Get-Content ./config/kibana.yml) -replace '#server.host: \"localhost\"', 'server.host: \"0.0.0.0\"' | Set-Content ./config/kibana.yml; `
    (Get-Content ./config/kibana.yml) -replace '#elasticsearch.url: \"http://localhost:9200\"', $('elasticsearch.url: \"{0}\"' -f $env:ELASTICSEARCH_URL) | Set-Content ./config/kibana.yml; `
    Add-Content ./config/kibana.yml 'path.data: d:\ ' | Set-Content ./config/kibana.yml

# Remove xpack
ENV XPACK_GRAPH_ENABLED=false `
    XPACK_ML_ENABLED=false `
    XPACK_MONITORING_ENABLED=false `
    XPACK_REPORTING_ENABLED=false `
    XPACK_SECURITY_ENABLED=false `
    XPACK_WATCHER_ENABLED=false `
    ES_JAVA_OPTS="-Xms512m -Xmx512m" `
    NODE_OPTIONS="--max-old-space-size=4096"

RUN './bin/kibana-plugin remove x-pack'

EXPOSE 5601

CMD [ "c:/kibana/bin/kibana.bat" ]

HEALTHCHECK --start-period=30s --interval=10s --retries=5 `
 CMD powershell -command `
    try { `
     $response = INVOKE-WEBREQUEST http://localhost:5601/app/kibana -UseBasicParsing; `
     if ($response.StatusCode -eq 200) { return 0} `
     else {return 1}; `
    } catch { return 1 }