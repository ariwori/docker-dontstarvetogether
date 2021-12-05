FROM yeetzone/gosu
MAINTAINER YEET Zone <support@yeet.zone>

# Set build arguments.
ARG STEAM_USER
ENV STEAM_USER ${STEAM_USER:-"steam"}
ARG STEAM_HOME
ENV STEAM_HOME ${STEAM_HOME:-"/opt/steam"}
ARG STEAM_URL
ENV STEAM_URL ${STEAM_URL:-"https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz"}

# Add our user/group first to ensure their IDs get set consistently.
RUN set -ex \
	&& groupadd -g 1001 -r $STEAM_USER \
	&& useradd -u 1001 -rm -d $STEAM_HOME -g $STEAM_USER $STEAM_USER

COPY /data/debian.sourcelist /etc/apt/sources.list

# Install everything.
RUN set -ex \
	# Install dependencies.
	&& apt-get update \
	&& apt-get install -y --no-install-recommends wget ca-certificates lib32gcc1 \
	# Install SteamCMD.
	&& wget -qO- $STEAM_URL | gosu $STEAM_USER tar -xzvC $STEAM_HOME \
	&& gosu $STEAM_USER $STEAM_HOME/steamcmd.sh \
		+@ShutdownOnFailedCommand 1 \
		+quit \
	&& rm -rf $STEAM_HOME/Steam/logs $STEAM_HOME/Steam/appcache/httpcache \
	&& find $STEAM_HOME/package -type f ! -name "steam_cmd_linux.installed" ! -name "steam_cmd_linux.manifest" -delete \
	# Clean up.
	&& apt-get purge -y --auto-remove wget \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Copy scripts.
COPY /scripts/steamcmd /usr/local/bin/steamcmd

# Copy entrypoint script.
COPY /entrypoint.sh /

# Set permissions.
RUN set -ex \
	&& chmod +x /entrypoint.sh \
	&& chmod +x /usr/local/bin/steamcmd

# Set entrypoint.
ENTRYPOINT ["/entrypoint.sh"]

# Set command.
CMD ["steamcmd"]


# Set environment.
ARG STEAM_PATH="/data"
ENV STEAM_PATH ${STEAM_PATH}

# Install dependencies.
RUN set -ex \
	&& dpkg --add-architecture i386 \
	&& apt-get update \
	&& apt-get install --yes --no-install-recommends \
		lib32stdc++6 \
		libcurl4-gnutls-dev:i386 \
	&& apt-get clean \
	&& rm -rf \
	    /tmp/* \
	    /var/tmp/* \
	    /var/lib/apt/lists/*


# Copy scripts.
COPY /scripts/dontstarve_dedicated_server_nullrenderer /usr/local/bin

# Set permissions.
RUN set -ex \
	&& mkdir -p $STEAM_PATH/bin \
	&& chown -R $STEAM_USER:$STEAM_USER $STEAM_PATH \
	&& chmod +x /usr/local/bin/dontstarve_dedicated_server_nullrenderer \
	&& sync

# Set working directory.
WORKDIR "$STEAM_PATH/bin"

# Set command.
CMD ["dontstarve_dedicated_server_nullrenderer"]


# Set build arguments.
ARG STORAGE_PATH="/opt/storage"
ENV STORAGE_PATH ${STORAGE_PATH}
ARG STEAM_APP="343050"
ENV STEAM_APP ${STEAM_APP}

# Set default environment.
ENV CLUSTER_NAME="cluster"
ENV SHARD_NAME="shard"
ENV SERVER_PORT="10999"
ENV SHARD_BIND_IP="0.0.0.0"
ENV ENCODE_USER_PATH=true
ENV BACKUP_LOG_COUNT=0

# Copy files.
COPY /scripts/dontstarvetogether /usr/local/bin/
COPY /scripts/ "$STEAM_HOME/scripts/"
COPY /data/ "$STEAM_HOME/data/"

# Set permissions.
RUN set -ex \
	&& mkfifo "$STEAM_HOME/console" \
	&& chmod -R +x "$STEAM_HOME/scripts/" \
	&& chmod +x /usr/local/bin/dontstarvetogether \
	&& sync

# Set up healthcheck.
HEALTHCHECK --start-period=15m --interval=5m --timeout=1m --retries=3 CMD dontstarvetogether version --check

# Set volumes.
VOLUME "$STORAGE_PATH"

# Set working directory.
WORKDIR "$STEAM_PATH/bin"

# Set command.
CMD ["dontstarvetogether", "start", "--update"]
