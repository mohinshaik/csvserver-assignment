# csvserver-assignment
**Part I:**
docker run -d infracloudio/csvserver:latest
docker ps -a
Check logs if it's failing:

bashCopydocker logs <container_id>

**Create gencsv.sh:**

bashCopy#!/bin/bash
start=$1
end=$2
for i in $(seq $start $end); do
    echo "$i, $((RANDOM % 1000))" >> inputFile
done

Make it executable and run:

bashCopychmod +x gencsv.sh
./gencsv.sh 2 8

Run container with inputFile:

bashCopydocker run -d -v $(pwd)/inputFile:/csvserver/inputdata infracloudio/csvserver:latest

Access shell and find port:

bashCopydocker exec -it <container_id> /bin/bash
netstat -tuln
Note the port, then exit and stop the container.

Run with correct port and environment variable:

bashCopydocker run -d -p 9393:9300 -v $(pwd)/inputFile:/csvserver/inputdata -e CSVSERVER_BORDER=Orange infracloudio/csvserver:latest

Create README.md with all commands executed.
Save the last docker run command to part-1-cmd file.
Generate output:

bashCopycurl -o ./part-1-output http://localhost:9393/raw

Generate logs:

bashCopydocker logs container_name > & part-1-logs

Commit and push changes to GitHub.

**Part II:**

Stop and remove all containers:

bashCopydocker stop $(docker ps -aq)
docker rm $(docker ps -aq)

Create docker-compose.yaml:

yamlCopyversion: '3'
services:
  csvserver:
    image: infracloudio/csvserver:latest
    ports:
      - "9393:9300"
    volumes:
      - ./inputFile:/csvserver/inputdata
    env_file:
      - csvserver.env

**Create csvserver.env:**

CopyCSVSERVER_BORDER=Orange

Test with:

bashCopydocker-compose up -d

Commit and push changes.

**Part III:**

Stop containers:

bashCopydocker-compose down

Update docker-compose.yaml:

yamlCopyversion: '3'
services:
  csvserver:
    image: infracloudio/csvserver:latest
    ports:
      - "9393:9300"
    volumes:
      - ./inputFile:/csvserver/inputdata
    env_file:
      - csvserver.env

  prometheus:
    image: prom/prometheus:v2.45.2
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

**Create prometheus.yml:**

yamlCopyglobal:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'csvserver'
    static_configs:
      - targets: ['csvserver:9300']

**Start services:**

bashCopydocker-compose up -d

Verify Prometheus at http://localhost:9090
Check csvserver_records metric in Prometheus.


