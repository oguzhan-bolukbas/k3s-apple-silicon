# k3s-apple-silicon
Multi-node k3s Cluster Installation on Apple Silicon MacOS with Multipass

## Installation Steps
### 1. Install QEMU:
QEMU is a must when using Multipass on Apple Silicon, because Multipass uses QEMU as the 
backend for virtualization on Apple Silicon.

```bash
brew install qemu
```

### 2. Install Multipass:
1. Download from here: https://canonical.com/multipass/download/macos

2. Verify Multipass is using QEMU:

```bash
multipass get local.driver
```
Should return:
`qemu`

### 3. Setup VMs with Multipass:
Cluster structure is:

| Purpose | Name | CPU | RAM | Disk |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| Load Balancer | lb | 1 | 512 M | 5 GB
| External DB | db | 2 | 2 G | 10 GB
| Master | master-1 | 4 | 8 G | 20 GB
| Master | master-2 | 4 | 8 G | 20 GB
| Master | master-3 | 4 | 8 G | 20 GB
| Worker | worker-1 | 4 | 4 G | 15 GB
| Worker | worker-2 | 4 | 4 G | 15 GB
| Worker | worker-3 | 4 | 4 G | 15 GB

<br> VM installation commands:
```bash
# Load Balancer
multipass launch --name lb --cpus 1 --memory 512M --disk 5G lts

# External DB
multipass launch --name db --cpus 2 --memory 2G --disk 10G lts

# Masters
multipass launch --name master-1 --cpus 4 --memory 8G --disk 20G lts
multipass launch --name master-2 --cpus 4 --memory 8G --disk 20G lts
multipass launch --name master-3 --cpus 4 --memory 8G --disk 20G lts

# Workers
multipass launch --name worker-1 --cpus 4 --memory 4G --disk 15G lts
multipass launch --name worker-2 --cpus 4 --memory 4G --disk 15G lts
multipass launch --name worker-3 --cpus 4 --memory 4G --disk 15G lts
```

List the installed VMs:

```bash
multipass list
```

The output must be looked like this:
```bash
Name                    State             IPv4             Image
db                      Running           192.168.205.5    Ubuntu 24.04 LTS
lb                      Running           192.168.205.4    Ubuntu 24.04 LTS
master-1                Running           192.168.205.6    Ubuntu 24.04 LTS
master-2                Running           192.168.205.7    Ubuntu 24.04 LTS
master-3                Running           192.168.205.8    Ubuntu 24.04 LTS
worker-1                Running           192.168.205.9    Ubuntu 24.04 LTS
worker-2                Running           192.168.205.10   Ubuntu 24.04 LTS
worker-3                Running           192.168.205.11   Ubuntu 24.04 LTS
```

### 4. Set Up External PostgreSQL DB
1. Shell into db:
```bash
multipass shell db
```

2. Install PostgreSQL:
```bash
sudo apt update
sudo apt install -y postgresql
```

3. Configure DB for external access:
```bash
sudo -u postgres psql
```

In the shell:
```bash
CREATE USER k3s WITH PASSWORD 'strongpassword';
CREATE DATABASE k3s OWNER k3s;
```

Exit with `\q`.

4. Learn PostgreSQL version:
```bash
ls /etc/postgresql
```

5. Edit pg_hba conf file:
```bash
sudo nano /etc/postgresql/<your_version>/main/pg_hba.conf
```

Add this end of the file:
```bash
host    all             all             0.0.0.0/0               md5
```

6. Edit postgresql conf file:
```bash
sudo nano /etc/postgresql/<your_version>/main/postgresql.conf
```

Uncomment this line:
```bash
#listen_addresses = 'localhost'         # what IP address(es) to listen on;
```

Then changed it to:
```bash
listen_addresses = '*'                   # what IP address(es) to listen on;
```

7. Restart the postgresql service:
```bash
sudo systemctl restart postgresql
```



