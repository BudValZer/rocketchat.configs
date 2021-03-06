FROM node:14.18.3-bullseye-slim

LABEL maintainer="buildmaster@rocket.chat"

RUN groupadd -g 65533 -r rocketchat && \
    useradd -u 65533 -r -g rocketchat rocketchat && \
    mkdir -p /app/uploads && \
    chown rocketchat:rocketchat /app/uploads

ADD --chown=rocketchat:rocketchat . /app

RUN apt-get update && \
    apt-get install -y --no-install-recommends fontconfig && \
    aptMark="$(apt-mark showmanual)" && \
    apt-get install -y --no-install-recommends g++ make python ca-certificates && \
    rm -rf /var/lib/apt/lists/* && \    
    cd /app/bundle/programs/server && \
    npm install && \
    apt-mark auto '.*' > /dev/null && \
    apt-mark manual $aptMark > /dev/null && \
    find /usr/local -type f -executable -exec ldd '{}' ';' | \
    awk '/=>/ { print $(NF-1) }' | sort -u | xargs -r dpkg-query --search | cut -d: -f1 | sort -u | xargs -r apt-mark manual && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false && \
    npm cache clear --force

USER rocketchat

VOLUME /app/uploads

WORKDIR /app/bundle

ENV DEPLOY_METHOD=docker \
    NODE_ENV=production \
    MONGO_URL=mongodb://mongo:27017/rocketchat \
    HOME=/tmp \
    PORT=3000 \
    ROOT_URL=http://localhost:3000 \
    Accounts_AvatarStorePath=/app/uploads

EXPOSE 3000

CMD ["node", "main.js"]