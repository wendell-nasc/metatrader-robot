FROM scottyhardy/docker-wine

ENV DEBIAN_FRONTEND noninteractive


ARG PYTHON_VERSION=3.7.9
ARG PYINSTALLER_VERSION=4.0

# we need wine for this to work, so we'll use the PPA
RUN apt-get update -qy \
    && apt-get install --no-install-recommends -qfy gpg-agent rename apt-transport-https software-properties-common cabextract wget curl zip unzip xvfb xdotool z xterm \    
    && apt-get update -qy \    
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* 
    

# wine settings
ENV WINEARCH win64
ENV WINEDEBUG fixme-all
ENV WINEPREFIX /wine

# PYPI repository location
ENV PYPI_URL=https://pypi.python.org/
# PYPI index location
ENV PYPI_INDEX_URL=https://pypi.python.org/simple

# install python in wine, using the msi packages to install, extracting
# the files directly, since installing isn't running correctly.

RUN set -x \
    && for msifile in `echo core dev exe lib path pip tcltk tools`; do \
        wget -nv "https://www.python.org/ftp/python/$PYTHON_VERSION/amd64/${msifile}.msi"; \
        wine msiexec /i "${msifile}.msi" /qb TARGETDIR=C:/Python37; \
        rm ${msifile}.msi; \
    done \
    && cd /wine/drive_c/Python37 \
    && echo 'wine '\''C:\Python37\python.exe'\'' "$@"' > /usr/bin/python \
    && echo 'wine '\''C:\Python37\Scripts\easy_install.exe'\'' "$@"' > /usr/bin/easy_install \
    && echo 'wine '\''C:\Python37\Scripts\pip.exe'\'' "$@"' > /usr/bin/pip \
    && echo 'wine '\''C:\Python37\Scripts\pyinstaller.exe'\'' "$@"' > /usr/bin/pyinstaller \
    && echo 'wine '\''C:\Python37\Scripts\pyupdater.exe'\'' "$@"' > /usr/bin/pyupdater \
    && echo 'assoc .py=PythonScript' | wine cmd \
    && echo 'ftype PythonScript=c:\Python37\python.exe "%1" %*' | wine cmd \
    && while pgrep wineserver >/dev/null; do echo "Waiting for wineserver"; sleep 1; done \
    && chmod +x /usr/bin/python /usr/bin/easy_install /usr/bin/pip /usr/bin/pyinstaller /usr/bin/pyupdater \
    && (pip install -U pip || true) \
    && rm -rf /tmp/.wine-*

ENV W_DRIVE_C=/wine/drive_c
ENV W_WINDIR_UNIX="$W_DRIVE_C/windows"
ENV W_SYSTEM64_DLLS="$W_WINDIR_UNIX/system32"
ENV W_TMP="$W_DRIVE_C/windows/temp/_$0"

# install Microsoft Visual C++ Redistributable for Visual Studio 2015, 2017 and 2019 dll files
RUN set -x \
    && rm -f "$W_TMP"/* \
    && wget -P "$W_TMP" https://aka.ms/vs/16/release/vc_redist.x64.exe \
    && cabextract -q --directory="$W_TMP" "$W_TMP"/vc_redist.x64.exe \
    && cabextract -q --directory="$W_TMP" "$W_TMP/a10" \
    && cabextract -q --directory="$W_TMP" "$W_TMP/a11" \
    && cd "$W_TMP" \
    && rename 's/_/\-/g' *.dll \
    && cp "$W_TMP"/*.dll "$W_SYSTEM64_DLLS"/

# install pyinstaller
RUN /usr/bin/pip install pyinstaller==$PYINSTALLER_VERSION

# put the src folder inside wine
RUN mkdir /src/ && ln -s /src /wine/drive_c/src
VOLUME /src/
WORKDIR /wine/drive_c/src/
RUN mkdir -p /wine/drive_c/tmp

COPY ./wine/entrypoint-windows.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]