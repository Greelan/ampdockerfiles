# Debian 13 with FEX image for AMP containers on ARM64
# cubecoders/ampbase:fex

FROM        debian:trixie-slim AS builder

ENV         DEBIAN_FRONTEND="noninteractive"

RUN         set -eux; \
            apt-get update; \
            apt-get install -o APT::Keep-Downloaded-Packages="false" --no-install-recommends -y \
                ca-certificates git cmake ninja-build pkgconf ccache clang llvm lld libssl-dev \
                python3-setuptools g++-x86-64-linux-gnu libgcc-14-dev-i386-cross libgcc-14-dev-amd64-cross \
                nasm python3-clang libstdc++-12-dev-amd64-cross libstdc++-12-dev-arm64-cross \
                squashfs-tools squashfuse libc-bin libc6-dev-i386-amd64-cross \
                lib32stdc++-14-dev-amd64-cross qtdeclarative5-dev qml-module-qtquick-controls \
                qml-module-qtquick-controls2 qml-module-qtquick-dialogs; \
            \
            git clone --recurse-submodules https://github.com/FEX-Emu/FEX.git; \
            cd FEX; \
            mkdir Build; \
            cd Build; \
            CC=clang CXX=clang++ cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DUSE_LINKER=lld -DENABLE_LTO=True -DBUILD_TESTING=False -DENABLE_ASSERTIONS=False -G Ninja ..; \
            ninja
 
FROM        debian:trixie-slim

LABEL       org.opencontainers.image.source="https://github.com/CubeCoders/dockerfiles"
LABEL       org.opencontainers.image.licenses=MIT

ENV         AMP_CONTAINER="1"
ENV         DEBIAN_FRONTEND="noninteractive"

COPY        --from=builder /FEX/Build/Bin/* /usr/bin/

# Update base packages and install dependencies
RUN         set -eux; \
            mkdir -p /usr/share/man/man1; \
            apt-get update; \
            apt-get install -o APT::Keep-Downloaded-Packages="false" -y \
                ca-certificates curl wget tar unzip xz-utils bzip2 \
                coreutils procps iproute2 iputils-ping socat jq git git-lfs gnupg tmux dbus \
                tini tzdata locales gosu \
                libssl3t64 libcurl4t64 libsqlite3-0 libzstd1 libsdl2-2.0-0 libsdl1.2debian libfontconfig1 libicu76 \
                # Required for Proton
                python3 \
                # Required for Core Keeper
                xvfb xauth libxi6 \
                # Required for Valheim crossplay (and variously some others)
                libc6 libatomic1 libpulse0 libpulse-mainloop-glib0 \
                # Required for BeamMP
                liblua5.3-0 \
                # Required for Eco
                libgdiplus \
                # Required for Pavlov VR (and variously some others)
                gdb libc++1 libc++abi1 libunwind8 libgcc-s1; \
            \  
            dpkg --add-architecture armhf; \
            apt-get update; \
            apt-get install -o APT::Keep-Downloaded-Packages="false" -y \
                # Required for steamcmd
                libgcc-s1:armhf \
                # Others
                libstdc++6:armhf zlib1g:armhf libbz2-1.0:armhf libcurl4t64:armhf libcurl3t64-gnutls:armhf \
                libncurses6:armhf libtinfo6:armhf libsdl2-2.0-0:armhf libssl3t64:armhf; \
            \
            # Install FEX dependencies and RootFS
            apt-get install -o APT::Keep-Downloaded-Packages="false" --no-install-recommends -y \
                squashfs-tools squashfuse; \
            \
            #install -d /tmp/.fex-emu/RootFS; \
            #printf '{"Config":{"RootFS":"Ubuntu_24_04.sqsh"},"ThunksDB":{}}' > /tmp/.fex-emu/Config.json; \
            #wget -qP /tmp/.fex-emu/RootFS https://rootfs.fex-emu.gg/Ubuntu_24_04/2025-03-04/Ubuntu_24_04.sqsh; \
            \
            # Temp fix if libssl1.1 needed
            wget -qO libssl1.1.deb https://ports.ubuntu.com/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.24_arm64.deb; \
            apt-get install -y ./libssl1.1.deb; \
            rm libssl1.1.deb; \
            \
            # Install AMP instance manager
            wget -q https://cdn-repo.c7rs.com/aarch64/ampinstmgr-latest.tgz; \
            tar -xzf ampinstmgr-latest.tgz -C /; \
            rm ampinstmgr-latest.tgz; \
            \
            # Set up locales
            sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen; \
            locale-gen; \
            \
            apt-get clean; \
            rm -rf /var/lib/apt/lists/*

ENV         LANG="en_US.UTF-8" LANGUAGE="en_US:en" LC_ALL="en_US.UTF-8"

STOPSIGNAL  SIGINT

COPY        ./scripts/base/ampstart.sh /ampstart.sh
RUN         chmod +x /ampstart.sh
ENTRYPOINT  ["/usr/bin/tini", "-g", "--", "/ampstart.sh"]
CMD         []
