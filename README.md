# bind-docker
This repository contains a Dockerized setup of the BIND DNS server, providing both internal and external views. The setup allows you to quickly deploy a DNS server with a customized configuration using Docker. Below is a guide on how to use this repository.

# Prerequisites
Docker and Docker Compose installed on your machine.
Basic understanding of DNS and BIND configuration.

# View Mechanism
In this BIND configuration, the DNS server is set up with two views: internal and external. Views allow the DNS server to provide different responses to queries based on the client's IP address or TSIG key.

- `Internal View`: This view serves clients within specified private IP ranges (e.g., 10.0.0.0/8, 192.168.0.0/16) and is configured to allow recursion and updates from trusted internal sources.
- `External View`: This view is meant for clients outside the internal network and is more restrictive, typically disallowing recursion and updates.

If you don't need separate internal and external views, you can simplify the configuration by using only the internal view. You need to edit `named.conf.local` and `named.conf.options`.

# Getting Started

## 1. Clone the Repository
bash
Copy code
git clone https://github.com/yourusername/bind-docker.git
cd bind-docker

## 2. Configure the DNS Server
The configuration files are located in the volumes/conf/ directory. The key configuration files are:

`named.conf.options`: General BIND options.
`named.conf.local`: Local DNS zones and views.
`named.conf.default-zones`: Default zones for the internal view.

You can customize these files according to your network's needs.

## 3. Start the DNS Server
Use Docker Compose to start the BIND DNS server:

```bash
docker compose up -d
```

This command will start the BIND server in a detached mode.

## 4. Managing the Server
### Enter the Container
To interact with the BIND server, you can enter the Docker container:

```bash
docker exec -ti bind /bin/bash
```

### Check Server Status
To check the status of the BIND server and verify authentication:

``bash
rndc -k /etc/bind/rndc-int.key -s 127.0.0.1 -p 953 status
```

## 5. Create TSIG Keys
To create TSIG keys for secure communication between DNS servers:

```bash
tsig-keygen -a hmac-sha512 secret > volumes/conf/rndc-int.key
tsig-keygen -a hmac-sha512 secret > volumes/conf/rndc-ext.key
```

## 6. Adding Zones
You can create new DNS zones for both external and internal views.

### Example for External View
Create the zone file:

```bash
cat <<'EOF' > /var/cache/bind/example.com-external
$TTL 300    ; 5 minutes
example.com.            IN SOA    ns1.example.com. hostmaster.example.com. (
                2024080530 ; serial
                3600       ; refresh (1 hour)
                600        ; retry (10 minutes)
                86400      ; expire (1 day)
                300        ; minimum (5 minutes)
                )
            NS    ns1.example.com.
            NS    ns2.example.com.
$TTL 300    ; 5 minutes
ns1            A    8.8.8.8
ns2            A    1.1.1.1
EOF
```

Add the zone to BIND:

```bash
rndc -k /etc/bind/rndc-ext.key addzone example.com in external '{ type master; file "/var/cache/bind/example.com-external"; };'
```

### Example for Internal View
Create the zone file:

```bash
cat <<'EOF' > /var/cache/bind/example.com-internal
$TTL 300    ; 5 minutes
example.com.            IN SOA    ns1.example.com. hostmaster.example.com. (
                2024080530 ; serial
                3600       ; refresh (1 hour)
                600        ; retry (10 minutes)
                86400      ; expire (1 day)
                300        ; minimum (5 minutes)
                )
            NS    ns1.example.com.
            NS    ns2.example.com.
$TTL 300    ; 5 minutes
ns1            A    10.0.1.2
ns2            A    10.0.1.2
EOF
```

Add the zone to BIND:

```bash
rndc -k /etc/bind/rndc-int.key addzone example.com in internal '{ type master; file "/var/cache/bind/example.com-internal"; };'
```

## 7. Updating DNS Records
You can add or update DNS records using nsupdate.

### Interactive Mode
```bash
nsupdate -k /etc/bind/rndc-int.key
server 127.0.0.1
zone example.com
update add example.com 300 A 10.0.1.2
send
```

### Using a File
Create a file with the commands:

```bash
cat <<EOF > add_record.nsupdate
server 127.0.0.1
zone example.com
update add example.com 300 A 10.0.1.2
send
EOF
```

Run nsupdate with the file:

```bash
nsupdate -k /etc/bind/rndc-int.key add_record.nsupdate
```

## Ports
The following ports are exposed by the BIND server:

53 TCP/UDP: DNS queries.
953 TCP/UDP: RNDC (Remote Name Daemon Control) for managing the DNS server.

## Volumes
The following volumes are used:

./volumes/data:/var/cache/bind: Stores DNS zone files.
./volumes/conf:/etc/bind: Stores BIND configuration files.
./volumes/scripts:/app/: Stores custom scripts.

## Troubleshooting
If you encounter any issues, check the BIND logs by entering the container and inspecting the log files.

```bash
docker logs -f bind
```

# License
This project is licensed under the MIT License. See the LICENSE file for more details.

# Contributing
Contributions are welcome! Please submit a pull request or open an issue to discuss any changes or improvements.
For any questions or support, please contact [eminwux at famous-google-mail-service].

