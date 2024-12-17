# Machine Walkthrough - LordOfTheRoot

## 1. Network Configuration

**Host IP:** `10.38.1.112`  
**Interface Information (ifconfig):**

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.38.1.112  netmask 255.255.255.0  broadcast 10.38.1.255
        ether 08:00:27:0a:d2:1d  txqueuelen 1000  (Ethernet)
        RX packets 218  bytes 16824 (16.4 KiB)
        TX packets 262864  bytes 15775954 (15.0 MiB)

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 08:00:27:f6:b2:b7  txqueuelen 1000  (Ethernet)
        RX packets 2036  bytes 191781 (187.2 KiB)
        TX packets 151  bytes 15959 (15.5 KiB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
```

**Note:** Machine is running in a virtual environment with an **Internal Adapter**.

---

## 2. Scanning for Machines on the Network

Using `nmap` to scan the range `10.38.1.112-130`:

```bash
nmap 10.38.1.112-130
```

**Results:**

```
Nmap scan report for 10.38.1.117
Host is up (0.0011s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:53:5C:8C (Oracle VirtualBox virtual NIC)
```

**Observations:**
- A new machine was detected at **`10.38.1.117`**.
- **Port 22 (SSH)** is open.

---

## 3. Attempt to SSH into Detected Machine

Attempting SSH as root on `10.38.1.117`:

```bash
ssh root@10.38.1.117
```

**Result:**

```
Permission denied, please try again.
```

**Observations:**
- A message **"KNOCK FRIEND TO ENTER"** was displayed, indicating **Port Knocking**.
- The hint **"Easy as 1, 2, 3"** suggests ports `1`, `2`, and `3` need to be knocked.

---

## 4. Port Knocking

Using `nmap` to check ports `1`, `2`, and `3`:

```bash
nmap -r -nP -p1,2,3 10.38.1.117
```

**Results:**

```
PORT  STATE    SERVICE
1/tcp filtered tcpmux
2/tcp filtered compressnet
3/tcp filtered compressnet
```

---

## 5. Full Port Scan

After knocking, running a full port scan:

```bash
nmap -p- 10.38.1.117
```

**Results:**

```
PORT     STATE SERVICE
22/tcp   open  ssh
1337/tcp open  waste
```

**Observations:**
- A new port **1337/tcp** is now open.

---

## 6. Investigating Port 1337

### 6.1 Checking HTTP Content on Port 1337

Using `curl` to check the content at `http://10.38.1.117:1337`:

```bash
curl http://10.38.1.117:1337
```

**Output:**

```html
<html>
<img src="/images/iwilldoit.jpg" align="middle">
</html>
```

**Observations:**
- The page contains an image **`iwilldoit.jpg`**.

---

### 6.2 Downloading and Analyzing the Image

#### Step 1: Download the Image

```bash
ls
```

**Output:**

```
info.txt  iwilldoit.jpg
```

#### Step 2: Inspect Metadata with ExifTool

```bash
exiftool iwilldoit.jpg
```

**Output:**

```
ExifTool Version Number         : 13.00
File Name                       : iwilldoit.jpg
Directory                       : .
File Size                       : 37 kB
File Modification Date/Time     : 2024:12:17 20:54:13+05:30
File Access Date/Time           : 2024:12:17 20:54:14+05:30
File Inode Change Date/Time     : 2024:12:17 20:54:14+05:30
File Permissions                : -rwxrwx---
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 72
Y Resolution                    : 72
Image Width                     : 336
Image Height                    : 512
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 336x512
Megapixels                      : 0.172
```

**Observations:**
- Standard metadata; nothing unusual so far.

#### Step 3: Extract Strings from the Image

```bash
strings iwilldoit.jpg
```

**Output:**

```
JFIF
... (truncated for brevity)
Q}M#R2
q==- 
&[IS
idu>o
... ... ...
```

**Observations:**
- Nothing interesting & readable hidden information was identified in the strings.

---

## 7. Enumerating Directories

Using `dirb` to check for hidden directories on port `1337`:

```bash
dirb http://10.38.1.117:1337/
```

**Results:**

```
---- Scanning URL: http://10.38.1.117:1337/ ----
==> DIRECTORY: http://10.38.1.117:1337/images/                                                                                                              
+ http://10.38.1.117:1337/index.html (CODE:200|SIZE:64)                                                                                                     
+ http://10.38.1.117:1337/server-status (CODE:403|SIZE:293)                                                                                                
                                                                                                                                                            
---- Entering directory: http://10.38.1.117:1337/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
```

**Observations:**
- Found a listable directory **`/images/`**.
- **`index.html`** is accessible (Code 200).
- **`server-status`** is forbidden (Code 403).

---

(Continue the enumeration process here...)
