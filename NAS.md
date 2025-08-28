## Description

All information below is relevant to QNAP TS-x64 device. My version is`-8G` which means  it has 8GB by default

Hardware  spec: https://www.qnap.com/en-us/product/ts-264/specs/hardware/TS-264-8G.pdf

## Issues

### 13. List all QuMagie albums

Creating new album from QuMagie can be a nightmare. The code seemed not to validate the user input, and if the timerange is invalid (ie `1970/1/1-`), album can be still created, but, the next album listing will generate 500 errors! And from that point, QuMagie seems totally useless. I have tried to remove/reinstall QuMagie but the list of known albums still persists, the issue still persists (though list of all media sources are gone and I have to input again -- this is weird, because this list is handle by Multiple Media Console)

To drop the problematic album, it's to drop the  thing using `mysql` tool!

```
$ ssh -L 3306:localhost:3310 admin@has -fN
$ mysql -u root -psecret -P 3306 -e 'use s01; select * from pictureAlbumTable;'
$ mysql -u root -psecret -P 3306 -e 'use s01; delete from pictureAlbumTable where iPhotoAlbumId=10;'
```

It's interesting that the AlbumId is integer, but from QuMagie interface you see it's a string, something like `nUwqer` or alike.

### 12. raid1 rebuilding

Qnap forum: https://forum.qnap.com/viewtopic.php?t=10268

```
[~] # cat /proc/sys/dev/raid/speed_limit_min
50000
[~] # cat /proc/sys/dev/raid/speed_limit_max 
10000000
[~] # cat /proc/mdstat       
Personalities : [linear] [raid0] [raid1] [raid10] [raid6] [raid5] [raid4] [multipath] 
md1 : active raid1 sdb3[3] sda3[2]
      3897063936 blocks super 1.0 [2/1] [_U]
      [==>..................]  recovery = 12.6% (494663040/3897063936) finish=4386.2min speed=12928K/sec
      bitmap: 4/30 pages [16KB], 65536KB chunk
...

# after a while

[~] # cat /proc/mdstat 
Personalities : [linear] [raid0] [raid1] [raid10] [raid6] [raid5] [raid4] [multipath] 
md1 : active raid1 sdb3[3] sda3[2]
      3897063936 blocks super 1.0 [2/1] [_U]
      [====>................]  recovery = 22.8% (892321280/3897063936) finish=568.8min speed=88039K/sec
      bitmap: 2/30 pages [8KB], 65536KB chunk

[~] # echo 70000 >/proc/sys/dev/raid/speed_limit_min 
```

The first disk rebuilding is complete. after almost 2 days. Now the second one with a lot better thing. Of course, after I disable the `qnas_console_install` and adjust the speed limit

```
$ ssh  admin@nas cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid10] [raid6] [raid5] [raid4] [multipath] 
md1 : active raid1 sdb3[2] sda3[3]
      3897063936 blocks super 1.0 [2/1] [U_]
      [>....................]  recovery =  2.0% (77987840/3897063936) finish=478.1min speed=133113K/sec
      bitmap: 1/30 pages [4KB], 65536KB chunk

md322 : active raid1 sdb5[2] sda5[0]
      6702656 blocks super 1.0 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md256 : active raid1 sdb2[2] sda2[0]
      530112 blocks super 1.0 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md13 : active raid1 sdb4[128] sda4[129]
      458880 blocks super 1.0 [128/2] [UU______________________________________________________________________________________________________________________________]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md9 : active raid1 sdb1[128] sda1[129]
      530048 blocks super 1.0 [128/2] [UU______________________________________________________________________________________________________________________________]
      bitmap: 1/1 pages [4KB], 65536KB chunk
```

### 11. qnas_console_install

PS: Even if we delete the binary file drawPic, system will restore them during the next boot.
It's better to kill the process using some cron-based job.

TODO. This, with drawPic daemon, was consuming CPU at an abonormal rate. killing the process is safe. Weird.

```
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
24902     1 admin    S    14488  0.0   1 11.6 /usr/share/qnas_console_install/drawPic
```

### 10. Data transfer speed

When using `rsync` to transfer huge amount of data between this device and another laptop/PC, I've noticed a degration of the transfer speed, which was used to 65MB/s, to ~ 10MB/s.

1. It's turned out that the cable/plugging issue. I changed the cable connection between NAS and the reciever end, using `ethtool` to ensure that 1Gbps mode is on (`ethtool NIC`) at both sides. (Later one I can confirm this cable is broken; there is no problem with NICs.)
2. (PC) Then I've seen the `cpupower` has contributed a huge factor into the speed issue.
    
	1. (PC) The new configuration sets the CPU frequency to 800MHz, and that generated about 10-12MB/s even if both NICs are connected with 1Gbps bandwidth.
    2. (PC) Temporarily I configured cpupower to use the max speed if applicable, and the maximal speed 65MBs/ comes back.

The difference between 3400MHz vs 2700Mhz? it's 65MBs vs 38MBs seeing through rsync output!

1. PS1: The maxium throughput measured/seen from NAS device  is 75MB/s. Mergefs with passthrough=off.
2. PS2: Quite often `rsync` process terminates without a nice trace (not dmesg, not segmentation fault). It's likely an issue with the NIC on the pc. Error from rsync seems to indicate there was  an authentication error (weird). NAS device logs doesn't have an interesting events
3.  PS3: (NAS) Iowait is reported to 26.5% when rsync needs to fetch a lot of small files. When big files are transferred, iolook up seems to be low (less iowait) and the speed could go up to  75MBs/
4. PS4: NAs reports disk throughput max to 66.9MB/s - 67.3MB/s

### 9. Adding new cronjob

1. Modify configuration (crontab-format): `/etc/config/crontab`
2. Reload/restart the daemon: `/etc/init.d/crond.sh reload`

### 8. Allow some users to ssh to system

tl;dr: Giving non-administrator access to the system is too dangerous. Because all folders on NAS system has very permissive permissions `777`, and well, you can add, remove do whatever on them QNAP host. 

```
$ cd /etc/config/ssh
$ {  echo AllowTcpForwarding yes; echo AllowUsers admin foo bar; } > sshd_user_config
$ # the file sshd_user_config will be used to generate the final configuration file /etc/config/ssh/sshd_config
$ # test configuration
$ /usr/sbin/sshd -t -f /etc/config/ssh/sshd_config -p 22
Unable to load host key "/etc/ssh/ssh_host_dsa_key": unknown or unsupported key type
Unable to load host key: /etc/ssh/ssh_host_dsa_key
$ :: /etc/init.d/services restart # well, restart all services ; don't take this massive action
$ :: /etc/init.d/login.sh restart # no this is not safe ! ssh won't be back

# Cross check, by dumping all active ssh configuration
$ sshd -T -f /etc/config/ssh/sshd_config
```

Go to  `Control Panel / Network and File services / Telnet + SSH / Disable + Enable`. There is place with `Edit Access Permission` but that doesn't allow you to include non-administrative users into the list. *Effectively QNAP has some custom patch so that the configuration file doesn't work?* From this page `Edit Access Permission`, it's mentioned that `only administrators can login remotely`

Please do note only a few algorithms are acceptable. From configuration:

```
PubkeyAcceptedAlgorithms +ssh-rsa-cert-v01@openssh.com,ssh-rsa
CASignatureAlgorithms +ssh-rsa
HostbasedAcceptedAlgorithms +ssh-rsa-cert-v01@openssh.com,ssh-rsa
```

Via `ssh -vv foo@nas_ip`:

```
debug2: host key algorithms: ssh-ed25519-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh
.com,sk-ssh-ed25519-cert-v01@openssh.com,sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com,rsa-sha2-256-cert-v01@openssh.com,ssh-ed25519,ecdsa-sha2-nist
p256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,sk-ssh-ed25519@openssh.com,sk-ecdsa-sha2-nistp256@openssh.com,rsa-sha2-512,rsa-sha2-256
```

Adding `AllowGrousp` to `sshd_user_config` doesn't work: The final compiled file doesn't include such setting. That means the generator script has intentionally overwritten user configuration!

The actual configuration file is found from `/mnt/HDA_ROOT/.config/...`

I have looked if there is PAM setting, no such special thing in /etc/pam.d. Now hard-coded debug level!

```
## Configure QuFirewall to allow port 33 to my client ip.
/usr/sbin/sshd -D -e -o UsePAM=no -f /etc/config/ssh/sshd_config -o LogLevel=debug -p 33

Could not open user 'foo' authorized keys '/share/homes/foo/.ssh/authorized_keys': Permission denied
debug1: restore_uid: 0/0
debug1: temporarily_use_uid: 1003/100 (e=0/0)
debug1: trying public key file /share/homes/foo/.ssh/authorized_keys2
Could not open user 'foo' authorized keys '/share/homes/foo/.ssh/authorized_keys2': Permission denied
debug1: restore_uid: 0/0
```

What the FilePermission here?! I have ensured unix permissions (`chmod/chown/getfacl/setfacl`) are correctly set up. but FileStation (web ui) doesn't have any trivial way to gain information about this path. So the issue is that the top / parent foler belongs to `adminisrators` group~

```
[/share] # ls -lad /share/CE_CACHEDEV1_DATA/homes/     
drwxrwxrwx 9 admin administrators 4096 2024-12-10 16:36 /share/CE_CACHEDEV1_DATA/homes//
```

I've changed the user home directory to `/home/_foo/`, fix all permissions, and get something new now

```
debug1: temporarily_use_uid: 1003/100 (e=0/0)
debug1: trying public key file /home/_foo//.ssh/authorized_keys
debug1: fd 6 clearing O_NONBLOCK
debug1: restore_uid: 0/0
debug1: temporarily_use_uid: 1003/100 (e=0/0)
debug1: trying public key file /home/_foo//.ssh/authorized_keys2
debug1: fd 6 clearing O_NONBLOCK
debug1: restore_uid: 0/0
```

**HOWEVER**, anything under `/home/_foo` will not surve a device reoobt. Those are just gone!

### 7. IGNORE_EXTENSIONS in Network Recycle Bin

Since 2013: https://forum.qnap.com/viewtopic.php?t=84584

> Problem: On a new TS-469U-RP + firmware 4.0.4 or 4.0.5 the functionality from Control Panel -> Network Recycle Bin -> Exclude these file extensions: (case insensitive, separated by comma ',') tmp, temp, wtmp, blk, dat, qold, qtmp don't work. The .tmp files are moved in recycle bin.

> Cause: IGNORE_EXTENSIONS option from /etc/config/libtrash.conf wants a semi-colon delimited list of file name extensions, not comma + space, like in UI.

> Temporary Solution: write in the input field from UI the extensions separated only by a semi-colon: tmp;temp;wtmp;blk;dat;qold;qtmp then click Apply button.
The list is saved OK in /etc/config/libtrash.conf, despite it is displayed wrong in the UI.

### 6. Nas upgrade interrupted and failed to boot

Written as much  as I still recall, 2025-06-20... My mistake not to have a write up quickly.

In the last day of Jan 2025 I decided to upgrade my NAS firmware. First I disabled network firewall  to allow the  device to connect to public internet, and  then process the normal upgrade.

It took so much time. The device seemed to boot up again, but got  stuck.  I decided to reboot it, didn't work either. After having my monitor pluggend into the hdmi port, I was shocked: The screen was black. something was terribly wrong. I turned off the device, and  booted up the device with an ArchLlinux usb. This was the cool part  of using Qnap: it's great chance to get some intial idea of the issue. What was that?  Well, the raid-1 array went wrong, and when reading  `/proc/mdstat`, it's clearly that the array was being rebuilt/recovered.

I spent the night to recover the raid array. This could be done by following the  basic instructions from ArchLinux's wiki page  https://wiki.archlinux.org/title/RAID. Nothing special, except that:

1. The raid-1 configuration wasn't assembled correctly at boot time. I expect sda-0 was going  with sdb-0, sda-1 with sdb-1 and so on, but in fact, the number went  quite  randomly. It took me a while to figure out  the correct assemble configuration
2. The sync process took very long time, I left them running and saw the  result (`/proc/mdstat`) on my TV (yes, the NAS was connected to the TV)

When I could confirm the raid array was healty status, I attempted to re-install the device
1. Unplug both disks and reset the  device: ttps://www.qnap.com/en/how-to/faq/article/how-can-i-reset-my-nas
2. Plug in 1 disk from the  raid-1 array and boot up the NAS
3. I can't find the link now, but  from what I read, it's possible that Installation wizard could recognize and use any existing disk/data. This was a good news and actually made me confident to process. And it's very true! During the very first steps of  the  installation Wizard, I carefully selected the option to re-use the existing disk.
4. After a while, the Nas was accessible, of course, with new UI password and certificates. Phew!
5. When I could access the NAS again, I did some basic things  to verity the health  of  the  disk. Then plugged the  2nd disk and waited for the system to recover the raid array.
6. The remained things is  to update admin password, reconfigured some docker setting (it seemed sth wrong happened, and I had to reconfigure my btsync docker daemon.)

Lessons learnt

1. Better to have backup of all data
2. Better to have backup of the whole NAS system (data and settings)
3. QNAP update  can  be painful, so can we just don't do that?

### 5. Prevent NAS from being updated a/o calling home

The following hosts are being used by internal QNAP services. To prevent the device from calling its home, some DNS tricky can be used. This is necessary and helpful when the outbound network has some low or limited quota.

- osm.api.myqnapcloud.com
- auth.myqnapcloud.io
- download.qnap.com
- update.qnap.com
- edge.myqnapcloud.io
- edge.myqnapcloud.io
- core2.myqnapcloud.io

### 4. Memory upgrade

qnap TS-x64 has two hardware (mainboard)  versions. The old version has two memory slots; that indicates it's possible to extend the device with  more memory. The discovery is found from this excellent thread: https://forum.qnap.com/viewtopic.php?t=170942

The information is availalbe via `demidecode`  command. According the output (below), the factory configuration comes with memory device `TS1GSH64V6B` which is  some kinda enterprise thing and you hardly find that on the normal market. Instead  I bought `TS2666HSB-8G` (Transcend DRAM module,  So-DIMM, DDR4 2666 1Rx8  (8G)) from Conrad (at €27.63) and it's  working well.

```
$ dmidecode |grep Memory -A 20
Physical Memory Array
	Location: System Board Or Motherboard
	Use: System Memory
	Error Correction Type: None
	Maximum Capacity: 32 GB
	Error Information Handle: Not Provided
	Number Of Devices: 2

Handle 0x003C, DMI type 19, 31 bytes
Memory Array Mapped Address
	Starting Address: 0x00000000000
	Ending Address: 0x003FFFFFFFF
	Range Size: 16 GB
	Physical Array Handle: 0x0039
	Partition Width: 2

Handle 0x003D, DMI type 221, 26 bytes
OEM-specific Type
	Header and Data:
		DD 1A 3D 00 03 01 00 08 07 22 30 00 02 00 00 00
		00 1D 00 03 00 01 0E 0F 00 00
	Strings:
		Reference Code - CPU
		uCode Version
		TXT ACM version

Handle 0x003E, DMI type 221, 26 bytes
OEM-specific Type
	Header and Data:
		DD 1A 3E 00 03 01 00 08 07 22 30 00 02 00 00 00
--
		MTRR (Memory type range registers)
		PGE (Page global enable)
		MCA (Machine check architecture)
		CMOV (Conditional move instruction supported)
		PAT (Page attribute table)
		PSE-36 (36-bit page size extension)
		CLFSH (CLFLUSH instruction supported)
		DS (Debug store)
		ACPI (ACPI supported)
		MMX (MMX technology supported)
		FXSR (FXSAVE and FXSTOR instructions supported)
		SSE (Streaming SIMD extensions)
		SSE2 (Streaming SIMD extensions 2)
		SS (Self-snoop)
		HTT (Multi-threading)
		TM (Thermal monitor supported)
		PBE (Pending break enabled)
	Version: Intel(R) Celeron(R) N5095 @ 2.00GHz
	Voltage: 1.1 V
	External Clock: 100 MHz
	Max Speed: 2900 MHz
--
Memory Device
	Array Handle: 0x0039
	Error Information Handle: Not Provided
	Total Width: 64 bits
	Data Width: 64 bits
	Size: 8 GB
	Form Factor: SODIMM
	Set: None
	Locator: Controller0-ChannelA
	Bank Locator: BANK 0
	Type: DDR4
	Type Detail: Synchronous
	Speed: 2667 MT/s
	Manufacturer: Transcend Information
	Serial Number: 00003767
	Asset Tag: 9876543210
	Part Number: TS1GSH64V6B         
	Rank: 1
	Configured Memory Speed: 2667 MT/s
	Minimum Voltage: 1.2 V
	Maximum Voltage: 1.2 V
	Configured Voltage: 1.2 V
	Memory Technology: DRAM
	Memory Operating Mode Capability: Volatile memory
	Firmware Version: Not Specified
	Module Manufacturer ID: Bank 2, Hex 0x4F
	Module Product ID: Unknown
	Memory Subsystem Controller Manufacturer ID: Unknown
	Memory Subsystem Controller Product ID: Unknown
	Non-Volatile Size: None
	Volatile Size: 8 GB
	Cache Size: None
	Logical Size: None

Handle 0x0047, DMI type 17, 100 bytes
Memory Device
	Array Handle: 0x0039
	Error Information Handle: Not Provided
	Total Width: 64 bits
	Data Width: 64 bits
	Size: 8 GB
	Form Factor: SODIMM
	Set: None
	Locator: Controller0-ChannelB
	Bank Locator: BANK 1
	Type: DDR4
	Type Detail: Synchronous
	Speed: 2667 MT/s
	Manufacturer: Transcend Information
	Serial Number: 00000050
	Asset Tag: 9876543210
	Part Number: TS2666HSB-8G        
	Rank: 1
	Configured Memory Speed: 2667 MT/s
	Minimum Voltage: 1.2 V
	Maximum Voltage: 1.2 V
	Configured Voltage: 1.2 V
	Memory Technology: DRAM
	Memory Operating Mode Capability: Volatile memory
	Firmware Version: Not Specified
	Module Manufacturer ID: Bank 2, Hex 0x4F
	Module Product ID: Unknown
	Memory Subsystem Controller Manufacturer ID: Unknown
	Memory Subsystem Controller Product ID: Unknown
	Non-Volatile Size: None
	Volatile Size: 8 GB
	Cache Size: None
	Logical Size: None

Handle 0x0048, DMI type 20, 35 bytes
Memory Device Mapped Address
	Starting Address: 0x00000000000
	Ending Address: 0x001FFFFFFFF
	Range Size: 8 GB
	Physical Device Handle: 0x0046
	Memory Array Mapped Address Handle: 0x003C
	Partition Row Position: Unknown
	Interleave Position: 1
	Interleaved Data Depth: 1

Handle 0x0049, DMI type 20, 35 bytes
Memory Device Mapped Address
	Starting Address: 0x00200000000
	Ending Address: 0x003FFFFFFFF
	Range Size: 8 GB
	Physical Device Handle: 0x0047
	Memory Array Mapped Address Handle: 0x003C
	Partition Row Position: Unknown
	Interleave Position: 2
	Interleaved Data Depth: 1

Handle 0x004A, DMI type 131, 64 bytes
OEM-specific Type
	Header and Data:
		83 40 4A 00 31 00 00 00 00 00 00 00 00 00 00 00
		F8 00 87 4D 00 00 00 00 01 00 00 00 32 00 0D 00
		18 05 0B 00 00 00 00 00 FE 00 FF FF 00 00 00 00
		00 00 00 00 20 00 00 00 76 50 72 6F 00 00 00 00

Handle 0x004B, DMI type 41, 11 bytes
Onboard Device
	Reference Designation: Onboard - Other
	Type: Other
	Status: Enabled
	Type Instance: 1
	Bus Address: 0000:00:00.0
...
```


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
