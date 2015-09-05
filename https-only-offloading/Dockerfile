FROM base:alpine

MAINTAINER Jorrit Salverda

# build time environment variables
ENV USER_NAME haproxy
ENV USER_ID 999
ENV GROUP_NAME haproxy
ENV GROUP_ID 999

# install haproxy
RUN addgroup -g $GROUP_ID -S $GROUP_NAME \
    && adduser -S -G $GROUP_NAME -u $USER_ID $USER_NAME \
    && apk --update add \
        haproxy \
    && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ADD haproxy.cfg /etc/haproxy/haproxy.cfg
ADD self-signed-certificate.pem /etc/ssl/private/self-signed-certificate.pem

# define mountable directories
VOLUME ["/etc/haproxy", "/etc/ssl/certs", "/etc/ssl/private/"]

# expose ports
EXPOSE 80 443

# Define working directory
WORKDIR /etc/haproxy

# runtime environment variables
ENV BACKEND_SERVER localhost
ENV BACKEND_SERVER_PORT 8153
ENV SSL_CERTIFICATE_NAME self-signed-certificate.pem

# define default command
CMD deluser $USER_NAME; delgroup $GROUP_NAME; addgroup -S -g $GROUP_ID $GROUP_NAME; adduser -S -G $GROUP_NAME -u $USER_ID $USER_NAME; sed -i -e "s/localhost:80/${BACKEND_SERVER}:${BACKEND_SERVER_PORT}/" /etc/haproxy/haproxy.cfg; sed -i -e "s/self-signed-certificate.pem/${SSL_CERTIFICATE_NAME}/" /etc/haproxy/haproxy.cfg; exec haproxy -db -f /etc/haproxy/haproxy.cfg