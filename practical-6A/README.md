

$$
\huge \begin{array}{c} \mathbf{\textsf{Royal University of Bhutan,}} \\ \mathbf{\textsf{College of Science and Technology}} \end{array}
$$

$$
\Large \mathbf{\textsf{DBS302: NoSQL Database Management }}
$$

$$
\Large \mathbf{\textsf{Computing Technologies Department  }}

$$

$$
\Large \mathbf{\textsf{Practical 6A  }}
$$

$$
\large \mathbf{\textsf{Submitted by: Sonam Dorji Ghalley  }}
$$

$$
\large \mathbf{\textsf{Student No: 02230299  }}
$$

## 1. Introduction

Security around data stands as a central challenge today, especially while firms shift toward NoSQL systems for handling private records. Through hands-on work, we examine how key protections - like login verification, data scrambling, and permission tiers shaped by user roles - function within two common platforms: Redis and MongoDB.

Holding data in memory, Redis often supports tasks like caching, managing user sessions, or tracking live activity. MongoDB works differently - storing information as documents, which helps when dealing with evolving or unstructured formats. Security does not come turned on completely at first, even if speed and strength seem solid right after setup. One must step in early to adjust access controls and encryption settings before putting either system into actual service. Without these steps, risks increase quickly once deployment begins outside test environments.

One key part of this exercise looks at basic security rules. Getting access means proving identity, often done by entering a correct name and secret code. While moving between systems, information stays shielded using special protection known as TLS. This method stops outsiders from reading what is being sent. Instead of giving everyone full control, people get only the permissions they need for their tasks. Such limits follow a strict rule: minimum necessary access. What matters most is matching rights to roles, not individuals.

Built step by step, this lab walks through activating Access Control Lists alongside password checks in Redis - then layers in TLS to protect data moving across connections. Instead of stopping there, it moves into MongoDB setup: users gain verified access, tailored permissions are assigned, encryption secures transit. Afterward, inspection takes place - not just checking boxes but looking closely at what holds, where gaps linger, what could still go wrong. From that look, suggestions emerge based on real findings, not assumptions. Each stage connects, yet stands apart in purpose and outcome.

Finishing this session means walking away with two protected databases. A methodical way to check their security emerges alongside, ready for real-world use. One step follows another until clarity takes shape. Through each task, attention shifts toward patterns that professionals rely on. Instead of random checks, structure guides the process. What results is both tangible and repeatable. Experience builds not by accident, but through design.

## 2. Aim

To configure and verify authentication, encryption, and role-based access control for Redis and MongoDB, and to perform a basic security audit of the configured databases.

## 3. Objective

By the end of this practical, the student should be able to:

1. Enable password-based authentication and Access Control Lists (ACL) in Redis.
2. Enable TLS encryption for Redis connections (at least at the configuration and client-connection level).
3. Enable authentication and RBAC in MongoDB using built-in user roles and custom roles.
4. Enable TLS for MongoDB so that all traffic is encrypted in transit.
5. Execute a simple security audit checklist (tests and commands) for both databases and record findings.

## 3. Theory

Authentication the process who checks if someone really is who they say they be? That happens before letting them touch a database. Redis uses passwords, along with something called **Access Control Lists** - first rolled out fully in version 6, then tweaked later in 7. Instead of one master key everyone shares, admins can now set up separate users, each with their own passcode. MongoDB turns on verification by flipping the security.authorization switch inside its config file. Once that's active, nobody does anything without handing over correct login details first. Leave it off, and both systems sit open to whoever finds the door on the network - that moment when trust becomes risk. So locking things down right away isn’t just smart - it’s where real protection begins.

When data moves between a client and a database, encryption during transfer keeps it safe from outside viewing or changes. Protection like this matters most if the connection runs across public or shared networks - places where messages can get intercepted. To block eavesdroppers, TLS guards that flow of information. For Redis, enabling secure links means pointing the config file to the right key and certificate locations, forcing all communication through an encrypted path. Rather than accepting open traffic, the system shuts out anything unencrypted once setup is complete. MongoDB takes a similar route - certificates go into mongod.conf so every exchange entering or leaving becomes shielded. Within practice setups, OpenSSL builds temporary certificates on demand instead of relying on official ones. These homemade files act just like the real thing while testing configurations ahead of live deployment. What you see here works exactly as it would later under actual trust chains. The stage mimics reality without needing external validation.

Security through roles means people get only what they need, nothing extra. Starting from zero trust, each role opens just enough doors. Commands in Redis stay locked unless someone has clearance by design. MongoDB handles this differently - roles come ready made, like read-only or full admin power. Custom jobs? Build new roles piece by piece. Access checks happen first, every time, before anything else runs. Encryption wraps data while moving and resting in place. Users prove who they are before any gate unlocks. Layered defenses stack up quietly, making breaches far less likely. Minimum access becomes the rule, not the exception.

## 4. Procedure

### Step 1 : Enable ACL and Create Users

starting the redis server with the new created redis.config file. 

- **NOTE:** I am using this new config file because i do not want to disturb my main configuration file and since this a practical i just want to simulate the it.

So the new config file that i created is `my-redis.config`  and inside the file i have the following 

```bash
# Redis config file on Desktop
bind 127.0.0.1
port 6379
protected-mode yes
daemonize no

# ACL Users
user default off
user admin on >adminStrongPwd ~* +@all
user app_user on >appStrongPwd ~session:* +get +set +del +expire +ttl
user monitoring on >monitorPwd ~* +@read +info

# Memory
maxmemory 256mb

# Logging
loglevel notice
logfile ""
```

- `on`: enables the user.
- `>password`: sets the password.
- `~pattern`: which key patterns the user can access.
- `+command` / `+@category`: which commands are allowed.

where in the new config file we are implementing ACL(access control list)

![image.png](Practical%206A/image.png)

starting the redis server with the following command 

```bash
redis-server my-redis.config
```

![image.png](Practical%206A/image%201.png)

starting the redis-cli now

```bash
redis-cli 
```

and now sending a ping request and usually we would be getting a `PONG`  response back now since we have used new config file we need to start the server using a password and username we would be getting the following

![image.png](Practical%206A/image%202.png)

now opening the redis-cli with the following command where ACL is implemented

connected as a admin:

```notion
redis-cli -a adminStrongPwd --user admin
```

![image.png](Practical%206A/image%203.png)

now we can see we are getting the `PONG`  response back from the server 

if we further explore based on the user login we do the following 

![image.png](Practical%206A/image%204.png)

connected as a user

```notion
redis-cli -a appStrongPwd --user app_user
```

![image.png](Practical%206A/image%205.png)

So therefore this a demonstration of ACL works in redis

### Step 2 : Enable TLS for Redis

In production, Redis should use TLS when crossing untrusted networks.

For the lab, a self-signed certificate can be used.

```bash
mkdir tls-certs
cd tls-certs
 
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt -subj "/C=BT/ST=Chukha/L=Phuntsholing/O=DBS302/OU=Lab/CN=redis-lab-ca"
 
openssl genrsa -out redis.key 4096
openssl req -new -key redis.key -out redis.csr -subj "/C=BT/ST=Chukha/L=Phuntsholing/O=DBS302/OU=Lab/CN=localhost"

openssl x509 -req -in redis.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out redis.crt -days 365 -sha256

# Create client key
openssl genrsa -out client.key 4096

# Create client certificate request
openssl req -new -key client.key -out client.csr -subj "/C=BT/ST=Chukha/L=Phuntsholing/O=DBS302/OU=Lab/CN=redis-client"

# Sign client certificate with your CA
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256
```

This creates `ca.crt`, `redis.crt`, `redis.key` in `/tls-certs` and in my case I’m creating it in the `Desktop/tls-certs` 

![image.png](Practical%206A/image%206.png)

so now add the following lines in the `my-redis.config`  

```bash
# Disable plain TCP if possible, or keep for demo only
port 0

# Enable TLS port
tls-port 6379

# TLS configuration
tls-ca-cert-file /etc/redis/tls/ca.crt
tls-cert-file /etc/redis/tls/redis.crt
tls-key-file /etc/redis/tls/redis.key

# Force clients to use TLS
tls-auth-clients yes
```

Restarting the redis-server after updating the above line

```bash
redis-server my-redis.config
```

![image.png](Practical%206A/image%207.png)

Now connecting using TLS (redis-cli) with the command:

```bash
redis-cli --tls \
  --cacert ~/Desktop/tls-certs/ca.crt \
  --user admin \
  --pass adminStrongPwd \
  -p 6379
```

![image.png](Practical%206A/image%208.png)

## Conclusion

A hands-on example showed Redis security using Access Control Lists alongside passwords and TLS encryption. Instead of relying on defaults, separate users got unique setups - distinct passwords, limited command access, key filters shaping their roles. One user handled just session: entries, another could view data but not alter it, while the third kept broad authority. Each setup matched actual scenarios where systems divide database needs by function. Clear boundaries emerged naturally when access followed responsibility.

Encryption during transfer got a boost when TLS settings locked down the Redis setup, shielding every bit moving between user and system. With keys made via OpenSSL, acting like real-world conditions, the test environment showed only secure links lived - others died silent. Fake certs did their job; attempts without encryption hit a wall, yet those with proper handshake sailed through. Data stayed safe because unprotected talks never reached the door, while verified ones passed clean. Security tightened as each message traveled cloaked, leaving eavesdroppers blind to the flow.

Most of the time, Part A made it clear - security in Redis does not come ready to go. Instead, someone has to set it up on purpose. Access control works well when shaped around what users actually need, nothing more. Encryption during data travel stops snooping at the wire level. From here, better login codes help. So do personal digital passes on the user end for tighter handshakes. Locking connections down to specific addresses adds another wall too.