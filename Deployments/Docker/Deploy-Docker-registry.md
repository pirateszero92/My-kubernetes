Create Directory

  mkdir docker-registry && cd docker-registry
  mkdir cert

put crt and key file in the cert folder.

Create docker-compose file

  nano docker-compose.yaml

Add to line :

  version: '3'
  
  services:
      docker-registry:
          container_name: k8s-nfs.local
          image: registry
          ports:
              - 443:443
          environment:
            - REGISTRY_HTTP_ADDR=0.0.0.0:443
            - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt 
            - REGISTRY_HTTP_TLS_KEY=/certs/server.key 
          restart: always
          volumes:
              - ./volume:/var/lib/registry
              - ./certs:/certs
      docker-registry-ui:
          container_name: docker-registry-ui
          image: konradkleine/docker-registry-frontend:v2
          ports:
              - 8443:443
          environment:
              ENV_DOCKER_REGISTRY_HOST: k8s-nfs.local
              ENV_DOCKER_REGISTRY_PORT: 443
              ENV_DOCKER_REGISTRY_USE_SSL: 1
              ENV_USE_SSL: 1
          volumes:
             - ./certs/server.crt:/etc/apache2/server.crt:ro
             - ./certs/server.key:/etc/apache2/server.key:ro


ref: https://github.com/kubernetesway/DevOps/tree/main
