# Usage

To run this docker container use the following command

```sh
docker run -d jsalverda/haproxy:https-only-offloading
```

# Environment variables

In order to configure the haproxy load balancer for providing ssl on port 443 for any backend server you can use the following environment variables

| Name                 | Description                                               | Default value               |
| -------------------- | ----------------------------------------------------------| --------------------------- |
| BACKEND_SERVER       | The ip address of the backend server                      | localhost                   |
| BACKEND_SERVER_PORT  | The http port the backend server listens to               | 80                          |
| SSL_CERTIFICATE_NAME | The pem filename for the ssl certificate used on port 443 | self-signed-certificate.pem |

To run haproxy to redirect to ssl and provide access through regular https port (443) to the backend server run the following command

```sh
docker run -d \
    -e "BACKEND_SERVER=backend-origin.yourdomain.com" \
    -e "BACKEND_SERVER_PORT=80" \
    -e "SSL_CERTIFICATE_NAME=www.yourdomain.com.pem" \
    jsalverda/haproxy:https-only-offloading
```

# Mounting volumes

In order to keep your ssl certificate outside of the container on the host machine you can mount the following directories

| Directory         | Description               | Importance                                                           |
| ----------------- | ------------------------- | -------------------------------------------------------------------- |
| /etc/haproxy      | Configuration for haproxy | If configuration needs to be different from the one in the container |
| /etc/ssl/certs    | CA certificates           | Keep these files safe                                                |
| /etc/ssl/private/ | SSL certificates          | Keep these files safe                                                |

Start the container like this to mount the directories

```sh
docker run -d \
    -e "BACKEND_SERVER=backend-origin.yourdomain.com" \
    -e "BACKEND_SERVER_PORT=80" \
    -e "SSL_CERTIFICATE_NAME=gocd.yourdomain.com.pem" \
    -v /mnt/persistent-disk/gocd-haproxy/config:/etc/haproxy
    -v /mnt/persistent-disk/gocd-haproxy/ssl-certs:/etc/ssl/certs
    -v /mnt/persistent-disk/gocd-haproxy/ssl-private:/etc/ssl/private
    jsalverda/haproxy:https-only-offloading
```

To make sure the process in the container can read and write to those directories create a user and group with same gid and uid on the host machine

```sh
groupadd -r -g 999 haproxy
useradd -r -g haproxy -u 999 haproxy
```

And then change the owner of the host directories

```sh
chown -R haproxy:haproxy /mnt/persistent-disk/gocd-haproxy/config
chown -R haproxy:haproxy /mnt/persistent-disk/gocd-haproxy/ssl-certs
chown -R haproxy:haproxy /mnt/persistent-disk/gocd-haproxy/ssl-private
```