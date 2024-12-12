# logstash-veeam-syslog Docker Container Setup

## Version Information
~~~~
Version: 0.4 (December 12 2024)
Author: Stephan "Steve" Herzig
~~~~

This guide will walk you through the steps to download, build, run, and make a persistent Logstash Docker container.

## Steps to Setup

### 1. Download the Files from GitHub

Clone the repository to your local machine:

```bash
git clone https://github.com/yetanothermightytool/logstash-veeam-syslog.git
```

### 2. Change into the Directory

Navigate into the project directory:

```bash
cd logstash-veeam-syslog
```

### 3. Adjust the Password for the `elastic` User

Ensure you replace the placeholder password `"Password"` with the actual password for the `elastic` user in the `logstash.conf` file.

In the `logstash.conf` file within the `pipeline` directory, locate the following section and update the `password` field:

```plaintext
output {
  elasticsearch {
    hosts => ["https://127.0.0.1:9200"]
    index => "veeam-vbr-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "<Password change here>"
    ssl_certificate_verification => false
  }
}
```

### 4. Select & Adjust the Pipeline Configuration

The default pipeline configuration file configures the container to listen on TCP port 5514. If you need to change this port, edit the pipeline configuration file before building the Docker image.

> **Note:** There is now a configuration file that extracts Veeam ONE Syslog messages in addition to the Veeam Backup & Replication Syslog messages (logstash-veeam-with-vone-syslog.conf).  Before building the container, ensure that only the configuration file you want to use is in the pipeline folder.

Open the pipeline configuration file:

```bash
vi pipeline/your_logstash_config.conf
```

Locate the section that defines the port (typically in the input section) and change it as needed:

```conf
input {
  tcp {
    port => 5514  # Change this to your desired port
  }
}
```

### 5. Build the Docker Image

Build the Docker image with the tag `logstash-veeam-syslog`:

```bash
sudo docker build -t logstash-veeam-syslog .
```

### 6. Run the Docker Container

Run the Docker container:

```bash
sudo docker run --network="host" -t logstash-veeam-syslog
```

## Making the Container persistent

To ensure the Docker container starts automatically on system boot, follow these steps:

### 1. Create a Systemd Service File

Open the systemd service file for editing:

```bash
sudo vi /etc/systemd/system/logstash-veeam-syslog-container.service
```

### 2. Add the Following Content to the File

```ini
[Unit]
Description=Logstash Container
Requires=snap.docker.dockerd.service
After=snap.docker.dockerd.service

[Service]
Restart=always
ExecStart=/snap/bin/docker run --network="host" -t logstash-veeam-syslog
ExecStop=/snap/bin/docker stop -t 2 logstash-veeam-syslog

[Install]
WantedBy=default.target
```

### 3. Reload Systemd

Reload systemd to recognize the new service file:

```bash
sudo systemctl daemon-reload
```

### 4. Enable the Service

Enable the service to start on boot:

```bash
sudo systemctl enable logstash-veeam-syslog-container.service
```

### 5. Start the Service

Start the service:

```bash
sudo systemctl start logstash-veeam-syslog-container.service
```

### 6. Check the Service Status

Verify that the service is running:

```bash
sudo systemctl status logstash-veeam-syslog-container.service
```

You should see output indicating that the service is active and running.

## Notes

This setup has been tested with:

- Ubuntu 22.04 LTS
- Logstash 8.14.0
- Docker installed via `snap`
- Veeam Backup & Replication 12.1.2, 12.2, 12.3
- Veeam ONE 12.3

## Version History
- 0.4
  - Veeam ONE Syslog messages can be extracted
- 0.3
  - Logstash configuration file update
  - Dockerfile points to v8.14.2
- 0.2
  - Initial version


**Please note this config is unofficial and is not created nor supported by Veeam Software.**
**The project is still a work in progress. The files stored in my GitHub can be used for an initial configuration.**
