###
<img width="800" height="448" alt="image" src="https://github.com/user-attachments/assets/a47f0e83-9033-474e-bb86-5d72c2ec5918" />
Prometheus → collects metrics

Node Exporter → exposes host metrics

cAdvisor → exposes container metrics

Grafana → visualizes everything

STEP 1: Prerequisites (Docker Host)
✅ Machine Requirements

Linux EC2 

Docker installed

Ports open:

9090 → Prometheus

3000 → Grafana

9100 → Node Exporter

8080 → cAdvisor

Install Docker on EC2 machine
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

STEP 2: What is Node Exporter & Why We Need It?
What is Node Exporter?

Node Exporter is a Prometheus exporter that exposes Linux host metrics like:

CPU usage

Memory usage

Disk I/O

Network traffic

What Node Exporter CANNOT do

It cannot see Docker containers

Only sees the host OS

That’s why we also need cAdvisor

STEP 3: Install Node Exporter (Docker Way)
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  --net=host \
  --pid=host \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter \
  --path.rootfs=/host

   Option                      Why we use it                                   
================================================== 
 `--net=host`               | Gives Node Exporter access to real host network 
 `--pid=host`               | Allows reading host process metrics             
 `-v "/:/host"`             | Mounts host filesystem                          
 `--path.rootfs=/host`      | Tells exporter where host root is                
 `--restart unless-stopped` | Auto-restart on failure                       

Verify
http://<HOST-IP>:9100/metrics

STEP 4: What is cAdvisor & Why We Need It?
What is cAdvisor?

cAdvisor (Container Advisor) collects:

Container CPU usage

Container memory usage

Container filesystem I/O

Running container count

Why Node Exporter is not enough?

Containers are not visible at OS level

cAdvisor reads from Docker runtime internals

   Mount              Purpose                        
==============================================
 `/var/lib/docker`  Container metadata             
 `/sys`             CPU & memory stats             
 `/var/run`         Docker socket                  
 `--privileged`     Required for low-level metrics 

 Verify
http://<HOST-IP>:8080

STEP 6: What is Prometheus & Why We Need It?
What is Prometheus?

Prometheus is a time-series metrics database that:

Scrapes metrics via HTTP

Stores them

Allows querying via PromQL

Why Prometheus?

Pull-based model (secure & scalable)

Native support for exporters

Industry standard

STEP 7: Create Prometheus Config
mkdir prometheus
cd prometheus

Create prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node-exporter"
    static_configs:
      - targets: ["<HOST-IP>:9100"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["<HOST-IP>:8080"]

 Why scrape interval?

15s = good balance between accuracy and load

STEP 8: Run Prometheus
docker run -d \
  --name prometheus \
  --restart unless-stopped \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

Verify
http://<HOST-IP>:9090/targets

STEP 9: What is Grafana & Why We Need It?
What is Grafana?

Grafana is a visualization layer:

Queries Prometheus

Displays metrics as dashboards

Supports alerts

Why Grafana?

Prometheus UI is raw

Grafana is human-readable

STEP 10: Run Grafana
docker run -d \
  --name grafana \
  --restart unless-stopped \
  -p 3000:3000 \
  grafana/grafana

  Login
http://<HOST-IP>:3000
username: admin
password: admin

STEP 11: Connect Grafana to Prometheus

Settings → Data Sources

Add → Prometheus

URL:

http://<HOST-IP>:9090

Save & Test
