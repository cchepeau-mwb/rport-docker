# rport-docker
repository to build a docker container for rport.

The new version doesn't have guacd inside the container as Apache switched from debian based image to Alpine linux for Guacd. Unfortunately Rport doesn't work on Alpine linux.

You will need to add a config file (preferably as a mounted read-only volume pointing to your local file)

If you want to use a database to store the data please follow this guide : https://oss.rport.io/get-started/api-authentication/

docker-compose
```
version: '3.9'
services:
  rport-server:
    container_name: rport
    image: cchepeaumwb/rport-docker:latest
    restart: unless-stopped
    privileged: true
    ports:
      - 3000:3000
      - 10000:8080
      - 20000-20100:20000-20100
    volumes:
      - /path/rportd.conf:/etc/rport/rportd.conf:ro
      - /path/rport.key:/var/lib/rport/rport.key:ro
      - /path/rport.crt:/var/lib/rport/rport.crt:ro
      - data:/var/lib/rport/
    
    command: bash -c "/usr/local/bin/rportd --data-dir /var/lib/rport -c /etc/rport/rportd.conf"
      
    healthcheck:
      test: wget --no-check-certificate --spider -S https://localhost:3000 2>&1 > /dev/null | grep -q "200 OK$"
      interval: 60s
      retries: 5
      start_period: 20s
      timeout: 10s

volumes:
  data:
