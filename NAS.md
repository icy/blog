## Issues 

### 3. QuFirewall is too permissive dealing with docker container

When turning on port forwarding for (any) container, the port is open by default and QuFirewall has no capacity to restrict access to those ports. The QuFirewall chains are different from the Docker chain. That means, the docker container's port is open to your local network and/or public domain in the  worse case.

Be careful and make sure your NAS hidden behind a blackbox firewall.

```
Chain DOCKER (1 references)
target prot opt source destination
ACCEPT udp -- 0.0.0.0/0 10.0.3.2 udp dpt:8889
ACCEPT tcp -- 0.0.0.0/0 10.0.3.2 tcp dpt:8888

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target prot opt source destination
DOCKER-ISOLATION-STAGE-2 all -- 0.0.0.0/0 0.0.0.0/0
RETURN all -- 0.0.0.0/0 0.0.0.0/0

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target prot opt source destination
DROP all -- 0.0.0.0/0 0.0.0.0/0
RETURN all -- 0.0.0.0/0 0.0.0.0/0

Chain DOCKER-USER (1 references)
target prot opt source destination
RETURN all -- 0.0.0.0/0 0.0.0.0/0
```

### 1. crypt-setup  and crypto module

When trying to open some external Luks-encrypted disk, I see some missing modules (as below.)
Contacting supporting team didn't help. The last message I got from the team was `Let me check with some guys. I will come back later.` And it's been 11 months I haven't seen any "come back".

```
~]# cryptsetup luksOpen /dev/sdb d2
Check that kernel supports serpent-xts-plain cipher

[~] # cryptsetup luksDump /dev/sdb
LUKS header information for /dev/sdb

Version: 1
Cipher name: serpent
Cipher mode: xts-plain
Hash spec: sha256
Payload offset: 4096
MK bits: 256

Key Slot 0: ENABLED
Iterations: 2226085
Key material offset: 8
AF stripes: 4000

RW, [12/1/23 6:38 PM]
Key Slot 1: ENABLED
Iterations: 2319858
Key material offset: 264
AF stripes: 4000
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

### 2. Support web site

Support site https://service.qnap.com/ doesn't work well behind squid proxy:
The site gives you some file to download instead of some redirected login page.
It's  probably due to squid configuration which fortunately works with all other sites.
