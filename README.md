# A Traefik setup for local development
A simple traefik setup for local development. Traefik is a cloud-native reverse proxy and load balancer built for 
microservices and containerized workloads.

## Usage
Clone the repo:
```
git clone https://github.com/JimCronqvist/traefik.git
```
Start the container:
```
docker-compose up -d
```
Access your application(s) via **http** at: http://localhost:3000 or any custom host you are using locally.

Access your application(s) via **https** at: https://localhost:3443 or any custom host you are using locally.

Access the Traefik dashboard at: http://localhost:9000

*Recommendation: Consider using a custom host of something like 'local.domain' in order to be able to generate
valid https certificates to use locally if required. That also has the benefit of easily enabling local DNS overrides.*

## Configure your docker-compose services to use traefik
Add the following labels to **each of the services** in the docker-compose file
```
labels:
  traefik.enable: true
  traefik.http.routers.[name].rule: Host(`my-service.localhost`) && PathPrefix(`/`)

  # If you don't have 'EXPOSE' in your Dockerfile, or if you have multiple ones, you need to set the port to use via a label.
  traefik.http.services.[name].loadbalancer.server.port: 3000
  
  # Optional label - only apply if you need to.
  # traefik.http.routers.[name].priority: 0
```
Replace the [name] placeholder with the name of the service in docker-compose, and update the rule for your setup.
For rule syntax see https://doc.traefik.io/traefik/routing/routers/#rule

Add the following section to your docker-compose file at the end to connect all services to the traefik network
```
networks:
  default:
    name: traefik
    external: true
```

Note that for existing setups, you will need to recreate the services **once** for it to work, by running `docker-compose up --force-recreate`

## For HTTPS usage (optional) - one-time setup

*Please note that this is only recommended if you are unable to generate a valid certificate for local development.
Or if you simply don't have a need for anything else. However, the self-signed traefik cert might be enough in that case.*

### Install mkcert
```
# Ubuntu
sudo apt install -y libnss3-tools

# Mac
brew install mkcert

# Windows
choco install -y mkcert
```

### Generate the CA and the certificate
```
mkcert -install
mkcert -cert-file certs/cert.pem -key-file certs/key.pem localhost 127.0.0.1 ::1 "*.localhost" local.[company].com "*.local.[company].com"
```

### Installing the root CA on other systems
Installing in the trust store does not require the CA key, so you can export the CA certificate and use mkcert to 
install it in other machines.

- Look for the rootCA.pem file in `mkcert -CAROOT`
- copy it to a different machine
- set `$CAROOT` to its directory
- run `mkcert -install`


### Installing the root CA on Mobile devices
See https://github.com/FiloSottile/mkcert?tab=readme-ov-file#mobile-devices
