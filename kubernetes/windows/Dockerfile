# Supported values of windows version are ltsc2019 or ltsc2022 which are being passed by the build script or build pipeline
ARG WINDOWS_VERSION=
FROM mcr.microsoft.com/windows/servercore:${WINDOWS_VERSION}
MAINTAINER OMSContainers@microsoft.com
LABEL vendor=Microsoft\ Corp \
    com.microsoft.product="Azure Monitor for containers"

ARG IMAGE_TAG=win-ciprod06142022

# Do not split this into multiple RUN!
# Docker creates a layer for every RUN-Statement
RUN powershell -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))"
# Fluentd depends on cool.io whose fat gem is only available for Ruby < 2.5, so need to specify --platform ruby when install Ruby > 2.5 and install msys2 to get dev tools
RUN choco install -y ruby --version 3.1.1.1 --params "'/InstallDir:C:\ruby31'" \
&& choco install -y msys2 --version 20211130.0.0 --params "'/NoPath /NoUpdate /InstallDir:C:\ruby31\msys64'" \
&& choco install -y vim

# gangams - optional MSYS2 update via ridk failing in merged docker file so skipping that since we dont need optional update
RUN refreshenv \
&& ridk install 3 \
&& echo gem: --no-document >> C:\ProgramData\gemrc \
&& gem install cool.io -v 1.7.1 --platform ruby \
&& gem install oj -v 3.3.10 \
&& gem install fluentd -v 1.14.6 \
&& gem install win32-service -v 1.0.1 \
&& gem install win32-ipc -v 0.7.0 \
&& gem install win32-event -v 0.6.3 \
&& gem install windows-pr -v 1.2.6 \
&& gem install tomlrb -v 2.0.1 \
&& gem install gyoku -v 1.3.1 \
&& gem sources --clear-all

# Remove gem cache and chocolatey
RUN powershell -Command "Remove-Item -Force C:\ruby31\lib\ruby\gems\3.1.0\cache\*.gem; Remove-Item -Recurse -Force 'C:\ProgramData\chocolatey'"

SHELL ["powershell"]

ENV tmpdir /opt/omsagentwindows/scripts/powershell

WORKDIR /opt/omsagentwindows/scripts/powershell

# copy certificate generator binaries zip
COPY ./omsagentwindows/*.zip /opt/omsagentwindows/

COPY setup.ps1 /opt/omsagentwindows/scripts/powershell
RUN ./setup.ps1

COPY main.ps1 /opt/omsagentwindows/scripts/powershell
COPY ./omsagentwindows/installer/scripts/filesystemwatcher.ps1 /opt/omsagentwindows/scripts/powershell
COPY ./omsagentwindows/installer/livenessprobe/livenessprobe.exe /opt/omsagentwindows/scripts/cmd/
COPY setdefaulttelegrafenvvariables.ps1 /opt/omsagentwindows/scripts/powershell

# copy ruby scripts to /opt folder
COPY ./omsagentwindows/installer/scripts/*.rb /opt/omsagentwindows/scripts/ruby/

# copy out_oms.so file
COPY ./omsagentwindows/out_oms.so /opt/omsagentwindows/out_oms.so

# copy fluent, fluent-bit and out_oms conf files
COPY ./omsagentwindows/installer/conf/fluent.conf /etc/fluent/
COPY ./omsagentwindows/installer/conf/fluent-bit.conf /etc/fluent-bit
COPY ./omsagentwindows/installer/conf/azm-containers-parser.conf /etc/fluent-bit/
COPY ./omsagentwindows/installer/conf/out_oms.conf /etc/omsagentwindows

# copy telegraf conf file
COPY ./omsagentwindows/installer/conf/telegraf.conf /etc/telegraf/

# copy keepcert alive ruby scripts
COPY ./omsagentwindows/installer/scripts/rubyKeepCertificateAlive/*.rb /etc/fluent/plugin/

#Copy fluentd ruby plugins
COPY ./omsagentwindows/ruby/ /etc/fluent/plugin/

ENV AGENT_VERSION ${IMAGE_TAG}
ENV OS_TYPE "windows"
ENV APPLICATIONINSIGHTS_AUTH "NzAwZGM5OGYtYTdhZC00NThkLWI5NWMtMjA3ZjM3NmM3YmRi"
ENV AZMON_COLLECT_ENV False
ENV CI_CERT_LOCATION "C://oms.crt"
ENV CI_KEY_LOCATION "C://oms.key"

ENTRYPOINT ["powershell", "C:\\opt\\omsagentwindows\\scripts\\powershell\\main.ps1"]
