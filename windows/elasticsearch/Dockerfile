# escape= `
FROM openjdk:8-jdk-nanoserver
SHELL ["powershell"]

# Elasticsearch
ENV ES_VERSION="6.1.1" `
    ES_HASH="58c90d988adac0b00ea4f2936aaae96fae63a2f00c2c1617efe53f86eccad669f92f7cec691dde81592ff3228922f98619289af103e125bf9c2ad6e1f7c16700" `
    ES_HOME="c:/elasticsearch" `
    ES_JAVA_OPTS="-Xms512m -Xmx512m" 

# install Elasticsearch
RUN Invoke-WebRequest $('http://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{0}.zip' -f $env:ES_VERSION) -OutFile 'elasticsearch.zip' -UseBasicParsing; `
    if ((Get-FileHash elasticsearch.zip -Algorithm sha512).Hash -ne $env:ES_HASH) { Write-Error 'Hash mismatch'; exit 1}; `
    Expand-Archive elasticsearch.zip -DestinationPath c:\ ; `
    Move-Item $('c:/elasticsearch-{0}' -f $env:ES_VERSION) $env:ES_HOME; `
    Remove-Item elasticsearch.zip

# On windows, Volumes get mapped via symlinks to directories on the
# host.  Some languages have problems resolving those symlinks in Docker.
# So...we map the volume to a logical drive, and then configure the app
# to use the logical drives.
# REF: https://github.com/moby/moby/issues/27537

# For this image that means:
#    We want a volume for es data directory
#    In the container, we use c:\es-data as the Volume
#    We map c:\es-data to D:\
#    We configure ES to write it's data to the D:
#    ditto for logs,   es-logs => e:

VOLUME ["c:/es-data"]
VOLUME ["c:/es-logs"]

RUN Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name ServerPriorityTimeLimit -Value 0 -Type DWord; `
    Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices'  -Name 'D:' -Value '\??\C:\es-data' -Type String; `
    Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices'  -Name 'E:' -Value '\??\C:\es-logs' -Type String

EXPOSE 9200 9300
WORKDIR c:\elasticsearch

# tweak the default config settings for docker
# this includes using the "D: in the docker container for the ES data directory"
RUN (Get-Content ./config/elasticsearch.yml) -replace '#network.host: 192.168.0.1', 'network.host: \"0.0.0.0\"' | Set-Content ./config/elasticsearch.yml; `
    (Get-Content ./config/elasticsearch.yml) -replace '#discovery.zen.minimum_master_nodes:', 'discovery.zen.minimum_master_nodes: 1' | Set-Content ./config/elasticsearch.yml; `
    (Get-Content ./config/elasticsearch.yml) -replace '#path.data: /path/to/data', 'path.data: d:\\' | Set-Content ./config/elasticsearch.yml; `
    (Get-Content ./config/elasticsearch.yml) -replace '#path.logs: /path/to/logs', 'path.logs: e:\\' | Set-Content ./config/elasticsearch.yml;

CMD [ "c:/elasticsearch/bin/elasticsearch.bat" ]

HEALTHCHECK --interval=5s `
CMD powershell -command `
    try { `
     $content = (iwr -useb http://localhost:9200/_cat/health).Content; `
     $health = $content.Split(' ')[3]; `
     if ($health -eq 'green' -or $health -eq 'yellow') { return 0 } `
     else { return 1 }; `
    } catch { return 1 }
