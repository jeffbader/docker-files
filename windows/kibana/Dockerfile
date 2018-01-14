# escape= `

FROM microsoft/windowsservercore
SHELL [ "powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Kibana
ENV KIBANA_VERSION="6.1.1"

RUN Invoke-WebRequest "https://artifacts.elastic.co/downloads/kibana/kibana-$($env:KIBANA_VERSION)-windows-x86_64.zip.sha512" -OutFile 'kibana.zip.sha512' -UseBasicParsing; `
    Invoke-WebRequest "https://artifacts.elastic.co/downloads/kibana/kibana-$($env:KIBANA_VERSION)-windows-x86_64.zip" -OutFile 'kibana.zip' -UseBasicParsing; `
    $kibanaHash = (Get-Content -Raw kibana.zip.sha512).Split(' ')[0].ToLower(); `
    if ((Get-FileHash kibana.zip -Algorithm sha512).Hash.ToLower() -ne $kibanaHash) {exit 1}; `
    Expand-Archive kibana.zip -DestinationPath C:\; `
    Move-Item c:/kibana-$($env:KIBANA_VERSION)-windows-x86_64 "C:\kibana"; `
    Remove-Item kibana.zip.sha512; `
    Remove-Item kibana.zip;

ENV ELASTICSEARCH_URL="http://elasticsearch:9200" 

WORKDIR C:/kibana

RUN (Get-Content ./config/kibana.yml) -replace '#server.host: \"localhost\"', 'server.host: \"0.0.0.0\"' | Set-Content ./config/kibana.yml; `
    (Get-Content ./config/kibana.yml) -replace '#elasticsearch.url: \"http://localhost:9200\"', $('elasticsearch.url: \"{0}\"' -f $env:ELASTICSEARCH_URL) | Set-Content ./config/kibana.yml;

EXPOSE 5601

HEALTHCHECK --start-period=30s --interval=10s --retries=5 `
 CMD powershell -command `
    try { `
     $response = INVOKE-WEBREQUEST http://localhost:5601/app/kibana -UseBasicParsing; `
     if ($response.StatusCode -eq 200) { return 0} `
     else {return 1}; `
    } catch { return 1 }

# CMD [ "c:/kibana/bin/kibana.bat" ]
CMD "./node/node.exe --max-old-space-size=10240 --trace-warnings ./src/cli"
