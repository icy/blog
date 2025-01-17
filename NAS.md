## Description

All information below is relevant to QNAP TS-x64 device. My version is`-8G` whic means  it has 8GB by default

Hardware  spec: https://www.qnap.com/en-us/product/ts-264/specs/hardware/TS-264-8G.pdf

## Issues 

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
