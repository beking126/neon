FROM neondatabase/build-tools:pinned

USER root
RUN set -e \
    && apt update \
    && apt install -y locales sudo \
    echo "root:control" | chpasswd \
    localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8


# Switch to nonroot user
USER nonroot:nonroot
WORKDIR /home/nonroot

ENV LANG en_US.utf8