# Builder container
FROM registry.redhat.io/ubi9/nodejs-18:1-118 AS build

# Install yarn
RUN npm install -g yarn -s &>/dev/null

# Copy app source
COPY . /opt/app-root/src/app
WORKDIR /opt/app-root/src/app

# Run install as supper tux
USER 0
RUN yarn install --frozen-lockfile --network-timeout 600000 && yarn build

# Web server container
FROM amazonlinux:2023

RUN INSTALL_PKGS="nginx findutils gettext" && \
    dnf install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all --enablerepo='*' && \
    mkdir -p /docker-entrypoint.d && \
    mkdir -p /etc/nginx/templates && \
    chown -R 1001:0 /var/lib/nginx /var/log/nginx /run /docker-entrypoint.d /etc/nginx/templates /etc/nginx/conf.d && \
    chmod -R ug+rwX /var/lib/nginx /var/log/nginx /run /docker-entrypoint.d /etc/nginx/templates /etc/nginx/conf.d

ENV NGINX_PORT=8080

# Use none-root user
USER 1001

# Set nginx configuration
# COPY nginx.conf /etc/nginx/nginx.conf

# When using ubi9/nginx-120 defaults:
#  listen       8080 default_server;
#  root         /opt/app-root/src;

COPY --from=build /opt/app-root/src/app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.conf.template /etc/nginx/templates/default.conf.template

COPY docker-entrypoint.sh /
COPY 20-envsubst-on-templates.sh /docker-entrypoint.d

ENTRYPOINT ["/docker-entrypoint.sh"]

# Run the server
CMD ["nginx", "-g", "daemon off;"]
