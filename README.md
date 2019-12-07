# การติดตั้งและเริ่มใช้งาน Hyperledger Fabric เบื้องต้น

*** ตัวอย่างนี้เป็นการติดตั้งและทดลองรัน Hyperledger Fabric version อย่างง่าย ซึ่งไม่เหมาะแ่กการใช้งานใน production environment ***

## Requirements

1. ติดตั้งโดยเครื่องมือ สคริปต์ และตัวอย่างที่ทาง official Hyperledger มีให้
2. ติดตั้งบนเครื่อง Stand alone เพียงเครื่องเดียว
3. ติดตั้งโดยใช้ Docker เป็นหลัก
4. ใช้ตัวอย่าง chaincode example_02 ที่ official มีให้
5. ผู้ติดตั้งมีสิทธิ์รัน docker

## System requirements

1. OS: Ubuntu 18.04 LTS (MacOS/Windows ก็สามารถติดตั้งได้เช่นกัน แต่ต้องใช้ทักษะ config Docker เพิ่มเติม)
2. CPU: ยิ่งเยอะยิ่งดี เพราะ Blockchain มี service หลายตัวมาก รวมไปถึงใช้งาน database หลาย instance
3. RAM: อย่างน้อย 8 GB สำหรับการติดตั้งแบบเครื่องเดียว
4. Internet: ไม่เช่นนั้นจะไม่สามารถ run docker container ได้

## Step โดยรวมในตัวอย่างนี้ประกอบด้วย

0. Setup เครื่องสำหรับใช้งาน notebook นี้
1. Setup เครื่องสำหรับการติดตั้งและใช้งาน Hyperledger Fabric
2. Generate crypto materail ซึ่งเป็น artifact สำหรับการสร้าง blockchain network และ permissioned channel สำหรับการทำ transaction  ต่างๆ
3. deploy blockchain network และทดสอบด้วย end to end test ที่ตัวอย่างให้มา
4. terminate blockchain service 

สำหรับเครื่องที่ใช้ทดสอบตัวอย่างนี้มีสเปคดังนี้


```bash
lsb_release -a
lscpu | grep "Model name"
lsmem | grep "Total online memory"
```

    No LSB modules are available.
    Distributor ID:	Ubuntu
    Description:	Ubuntu 18.04.3 LTS
    Release:	18.04
    Codename:	bionic
    [01;31m[KModel name[m[K:          Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
    [01;31m[KTotal online memory[m[K:       8G
    


# Step 0 Install & Setup environment สำหรับ notebook นี้

*** step นี้ให้ run แต่ละ command ใน terminal นะครับ ***

ติดตั้ง  [Bash kernel](https://github.com/Calysto/calysto_bash) สำหรับ Jupyter

```shell
pip install calysto_bash
```

แก้ให้ไม่ต้องใส่ password สำหรับคำสั่ง sudo ด้วยการ edit file /etc/sudoers

```shell
sudo visudo
```

ใส่

```text
username     ALL=(ALL) NOPASSWD:ALL
```

เข้าไปในบรรทัดสุดท้ายและ save file /etc/sudoers

เปิด notebook นี้ด้วยการรันคำสั่งใน terminal

```shell
sudo -E env "PATH=$PATH" jupyter notebook --allow-root
```

# Step 1 Setup เครื่องที่ใช้งาน

## ติดตั้ง docker engine และ docker-compose ต่างๆ


```bash
sudo apt-get install -y docker.io docker-compose
```

    
    
    
    docker-compose is already the newest version (1.17.1-2).
    docker.io is already the newest version (18.09.7-0ubuntu1~18.04.4).
    0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    


## ตั้งค่าให้ user สามารถใช้ docker ได้โดยไม่ต้องใช้ `sudo` ([Docker post installation](https://docs.docker.com/install/linux/linux-postinstall/))



```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

    groupadd: group 'docker' already exists
    


## ทดสอบการทำงานของ docker


```bash
docker run hello-world
```

    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.
    
    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash
    
    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/
    
    For more examples and ideas, visit:
     https://docs.docker.com/get-started/
    
    


# Step 2 Download Hyperledger Fabric binary files และ ตัวอย่าง

download file ที่จำเป็นในการใช้งาน โดยจะมีสิ่งที่ต้อง download 3 ประเภทหลัก คือ 

### 1. Hyperledger Fabric docker images
เป็น docker image ที่ pack core service ของ Hyperledger Fabric เอาไว้ ซึ่งสามารถ custom config/parameter ต่างๆได้
### 2. Hyperledger Fabric samples 
เป็นตัวอย่าง config และ chaincode (smart contract) ที่ทางทีมงาน Hyperledger เตรีมไว้ให้ทดลองเล่น และสามารถนำไปประยุกต์ใช้กับ app ของเราได้
### 3. Hyperledger Fabric platform-specific binaries
เป็น tool สำหรับการสร้าง crypto material และเรียกใช้งาน blockchain service ต่างๆ

คำสั่งที่ใช้ download ทุกอย่างแบบปกติ คือ

```shell
curl -sSL http://bit.ly/2ysbOFE | bash -s
```

step ข้างบนจะใช้เวลาค่อนข้างนาน เพราะต้อง download
1. Hyperledger Fabric docker images จำนวนมาก
2. Hyperledger Fabric samples หลายตัวอย่าง
3. Hyperledger Fabric platform-specific binaries ซึ่งใช้เวลา download ค่อนข้างนาน


เรามามารถ bypass ได้โดยการใส่ option

-d : bypass docker image download

-s : bypass fabric-samples repo clone

-b : bypass download of platform-specific binaries

ตัวอย่างการใช้งานเมื่อไม่ต้องการ download docker image และ samples

```shell
curl -sSL http://bit.ly/2ysbOFE | bash -s -- -d -s
```

ในตัวอย่างนี้จะทำการ download เฉพาะ binary file และตัวอย่างในเวอร์ชั่นเก่าเท่านั้น ซึ่ง docker image ที่จำเป็นจะถูก pull ในขณะ deploy service อยู่แล้ว


```bash
%%shell
## official scripts, however, i cannot download binaries with this script
# curl -sSL http://bit.ly/2ysbOFE | bash -s -- -d

# Changed script to my stable version
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/v1.4.3/scripts/bootstrap.sh | bash -s -- -d
```

    
    
    
    Installing hyperledger/fabric-samples repo
    
    
    
    
    
    ===> Checking out v1.4.3 of hyperledger/fabric-samples
    
    
    HEAD is now at f86ec95 [FAB-16390] Added filter for invalid transactions
    
    
    
    
    
    Installing Hyperledger Fabric binaries
    
    
    
    
    
    ===> Downloading version 1.4.3 platform specific fabric binaries
    
    
    ===> Downloading:  https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.4.3/hyperledger-fabric-linux-amd64-1.4.3.tar.gz
    
    
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    
    
                                     Dload  Upload   Total   Spent    Left  Speed
    
    
    
    
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:02 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:03 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:04 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:06 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:07 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:08 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:09 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:11 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:12 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:13 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:14 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:15 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:16 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:17 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:18 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:19 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:20 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:21 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:22 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:23 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:24 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:25 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:26 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:27 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:28 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:29 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:30 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:31 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:32 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:33 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:34 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:35 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:36 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:37 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:38 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:39 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:40 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:41 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:42 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:43 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:44 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:45 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:46 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:47 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:48 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:49 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:50 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:51 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:52 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:53 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:54 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:55 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:56 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:57 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:58 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:59 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:01:00 --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:01:00 --:--:--     0
    
    
    curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to nexus.hyperledger.org:443 
    
    
    
    
    
    gzip: stdin: unexpected end of file
    
    
    tar: Child returned status 1
    
    
    tar: Error is not recoverable: exiting now
    
    
    ==> There was an error downloading the binary file. Switching to incremental download.
    
    
    ==> Downloading file...
    
    
    ==> File downloaded. Verifying the md5sum...
    
    
    ==> Extracting hyperledger-fabric-linux-amd64-1.4.3.tar.gz...
    
    
    ==> Done.
    
    
    ===> Downloading version 1.4.3 platform specific fabric-ca-client binary
    
    
    ===> Downloading:  https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/linux-amd64-1.4.3/hyperledger-fabric-ca-linux-amd64-1.4.3.tar.gz
    
    
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    
    
                                     Dload  Upload   Total   Spent    Left  Speed
    
    
    
    
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    
      0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
    
      0 6382k    0 49396    0     0  17942      0  0:06:04  0:00:02  0:06:02 17936
    
      2 6382k    2  144k    0     0  37138      0  0:02:55  0:00:03  0:02:52 37129
    
      4 6382k    4  264k    0     0  58937      0  0:01:50  0:00:04  0:01:46 58924
    
      6 6382k    6  392k    0     0  72855      0  0:01:29  0:00:05  0:01:24 81919
    
      9 6382k    9  608k    0     0  92381      0  0:01:10  0:00:06  0:01:04  123k
    
     10 6382k   10  688k    0     0  91800      0  0:01:11  0:00:07  0:01:04  129k
    
     14 6382k   14  952k    0     0   110k      0  0:00:57  0:00:08  0:00:49  175k
    
     17 6382k   17 1088k    0     0   110k      0  0:00:57  0:00:09  0:00:48  157k
    
     20 6382k   20 1304k    0     0   124k      0  0:00:51  0:00:10  0:00:41  184k
    
     22 6382k   22 1456k    0     0   124k      0  0:00:51  0:00:11  0:00:40  172k
    
     25 6382k   25 1648k    0     0   127k      0  0:00:49  0:00:12  0:00:37  184k
    
     27 6382k   27 1776k    0     0   131k      0  0:00:48  0:00:13  0:00:35  167k
    
     30 6382k   30 1976k    0     0   134k      0  0:00:47  0:00:14  0:00:33  180k
    
     33 6382k   33 2120k    0     0   132k      0  0:00:48  0:00:15  0:00:33  147k
    
     36 6382k   36 2304k    0     0   136k      0  0:00:46  0:00:16  0:00:30  162k
    
     37 6382k   37 2416k    0     0   138k      0  0:00:46  0:00:17  0:00:29  166k
    
     40 6382k   40 2616k    0     0   139k      0  0:00:45  0:00:18  0:00:27  160k
    
     43 6382k   43 2752k    0     0   137k      0  0:00:46  0:00:19  0:00:27  148k
    
     44 6382k   44 2832k    0     0   136k      0  0:00:46  0:00:20  0:00:26  149k
    
     47 6382k   47 3000k    0     0   139k      0  0:00:45  0:00:21  0:00:24  151k
    
     50 6382k   50 3240k    0     0   142k      0  0:00:44  0:00:22  0:00:22  157k
    
     53 6382k   53 3416k    0     0   144k      0  0:00:44  0:00:23  0:00:21  162k
    
     55 6382k   55 3536k    0     0   143k      0  0:00:44  0:00:24  0:00:20  170k
    
     58 6382k   58 3720k    0     0   146k      0  0:00:43  0:00:25  0:00:18  187k
    
     61 6382k   61 3920k    0     0   146k      0  0:00:43  0:00:26  0:00:17  176k
    
     63 6382k   63 4064k    0     0   145k      0  0:00:43  0:00:27  0:00:16  157k
    
     66 6382k   66 4256k    0     0   149k      0  0:00:42  0:00:28  0:00:14  170k
    
     68 6382k   68 4392k    0     0   145k      0  0:00:43  0:00:30  0:00:13  154k
    
     71 6382k   71 4568k    0     0   147k      0  0:00:43  0:00:31  0:00:12  153k
    
     73 6382k   73 4680k    0     0   147k      0  0:00:43  0:00:31  0:00:12  150k
    
     75 6382k   75 4800k    0     0   146k      0  0:00:43  0:00:32  0:00:11  149k
    
     77 6382k   77 4928k    0     0   147k      0  0:00:43  0:00:33  0:00:10  136k
    
     81 6382k   81 5176k    0     0   149k      0  0:00:42  0:00:34  0:00:08  169k
    
     83 6382k   83 5304k    0     0   148k      0  0:00:42  0:00:35  0:00:07  159k
    
     85 6382k   85 5456k    0     0   149k      0  0:00:42  0:00:36  0:00:06  161k
    
     86 6382k   86 5520k    0     0   146k      0  0:00:43  0:00:37  0:00:06  152k
    
     90 6382k   90 5792k    0     0   150k      0  0:00:42  0:00:38  0:00:04  171k
    
     93 6382k   93 5936k    0     0   148k      0  0:00:42  0:00:39  0:00:03  146k
    
     95 6382k   95 6104k    0     0   150k      0  0:00:42  0:00:40  0:00:02  164k
    
     98 6382k   98 6280k    0     0   151k      0  0:00:42  0:00:41  0:00:01  166k
    
    100 6382k  100 6382k    0     0   151k      0  0:00:42  0:00:42 --:--:--  193k
    
    
    ==> There was an error downloading the binary file. Switching to incremental download.
    
    
    ==> Downloading file...
    
    
    ==> File downloaded. Verifying the md5sum...
    
    
    ==> Extracting hyperledger-fabric-ca-linux-amd64-1.4.3.tar.gz...
    
    ==> Done.
    


[ข้อมูลการติดตั้งเพิ่มเติม](https://hyperledger-fabric.readthedocs.io/en/release-1.4/install.html)

Set system path เพื่อให้ระบบมองเห็น binary


```bash
export PATH=$PWD/bin:$PATH
```

    


# Step 3 Generate crypto material

change directory ไปที่ fabric-samples/first-network


```bash
cd fabric-samples/first-network
```

    


ลองดูว่าใน script first network สามารถทำอะไรได้บ้าง


```bash
bash byfn.sh
```

    Usage: 
      byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-o <consensus-type>] [-i <imagetag>] [-a] [-n] [-v]
        <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
          - 'up' - bring up the network with docker-compose up
          - 'down' - clear the network with docker-compose down
          - 'restart' - restart the network
          - 'generate' - generate required certificates and genesis block
          - 'upgrade'  - upgrade the network from version 1.3.x to 1.4.0
        -c <channel name> - channel name to use (defaults to "mychannel")
        -t <timeout> - CLI timeout duration in seconds (defaults to 10)
        -d <delay> - delay duration in seconds (defaults to 3)
        -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
        -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
        -l <language> - the chaincode language: golang (default) or node
        -o <consensus-type> - the consensus-type of the ordering service: solo (default), kafka, or etcdraft
        -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
        -a - launch certificate authorities (no certificate authorities are launched by default)
        -n - do not deploy chaincode (abstore chaincode is deployed by default)
        -v - verbose mode
      byfn.sh -h (print this message)
    
    Typically, one would first generate the required certificates and 
    genesis block, then bring up the network. e.g.:
    
    	byfn.sh generate -c mychannel
    	byfn.sh up -c mychannel -s couchdb
            byfn.sh up -c mychannel -s couchdb -i 1.4.0
    	byfn.sh up -l node
    	byfn.sh down -c mychannel
            byfn.sh upgrade -c mychannel
    
    Taking all defaults:
    	byfn.sh generate
    	byfn.sh up
    	byfn.sh down
    


Generate crypto material สำหรับ channel ชื่อ `getstartfabric`


```bash
# enter y to continue
echo "y" | 
bash byfn.sh generate -c getstartfabric
```

    Generating certs and genesis block for channel 'getstartfabric' with CLI timeout of '10' seconds and CLI delay of '3' seconds
    proceeding ...
    /home/ratthapon/Blockchain/fabric-samples/first-network/../bin/cryptogen
    
    ##########################################################
    ##### Generate certificates using cryptogen tool #########
    ##########################################################
    + cryptogen generate --config=./crypto-config.yaml
    org1.example.com
    org2.example.com
    + res=0
    + set +x
    
    Generate CCP files for Org1 and Org2
    /home/ratthapon/Blockchain/fabric-samples/first-network/../bin/configtxgen
    ##########################################################
    #########  Generating Orderer Genesis block ##############
    ##########################################################
    CONSENSUS_TYPE=solo
    + '[' solo == solo ']'
    + configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
    [34m2019-12-08 06:28:24.059 +07 [common.tools.configtxgen] main -> INFO 001[0m Loading configuration
    [34m2019-12-08 06:28:24.115 +07 [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002[0m orderer type: solo
    [34m2019-12-08 06:28:24.115 +07 [common.tools.configtxgen.localconfig] Load -> INFO 003[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.165 +07 [common.tools.configtxgen.localconfig] completeInitialization -> INFO 004[0m orderer type: solo
    [34m2019-12-08 06:28:24.165 +07 [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 005[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.167 +07 [common.tools.configtxgen] doOutputBlock -> INFO 006[0m Generating genesis block
    [34m2019-12-08 06:28:24.167 +07 [common.tools.configtxgen] doOutputBlock -> INFO 007[0m Writing genesis block
    + res=0
    + set +x
    
    #################################################################
    ### Generating channel configuration transaction 'channel.tx' ###
    #################################################################
    + configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID getstartfabric
    [34m2019-12-08 06:28:24.192 +07 [common.tools.configtxgen] main -> INFO 001[0m Loading configuration
    [34m2019-12-08 06:28:24.245 +07 [common.tools.configtxgen.localconfig] Load -> INFO 002[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.299 +07 [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003[0m orderer type: solo
    [34m2019-12-08 06:28:24.299 +07 [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.299 +07 [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 005[0m Generating new channel configtx
    [34m2019-12-08 06:28:24.301 +07 [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 006[0m Writing new channel tx
    + res=0
    + set +x
    
    #################################################################
    #######    Generating anchor peer update for Org1MSP   ##########
    #################################################################
    + configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID getstartfabric -asOrg Org1MSP
    [34m2019-12-08 06:28:24.325 +07 [common.tools.configtxgen] main -> INFO 001[0m Loading configuration
    [34m2019-12-08 06:28:24.380 +07 [common.tools.configtxgen.localconfig] Load -> INFO 002[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.432 +07 [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003[0m orderer type: solo
    [34m2019-12-08 06:28:24.432 +07 [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.432 +07 [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005[0m Generating anchor peer update
    [34m2019-12-08 06:28:24.432 +07 [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006[0m Writing anchor peer update
    + res=0
    + set +x
    
    #################################################################
    #######    Generating anchor peer update for Org2MSP   ##########
    #################################################################
    + configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID getstartfabric -asOrg Org2MSP
    [34m2019-12-08 06:28:24.456 +07 [common.tools.configtxgen] main -> INFO 001[0m Loading configuration
    [34m2019-12-08 06:28:24.508 +07 [common.tools.configtxgen.localconfig] Load -> INFO 002[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.562 +07 [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003[0m orderer type: solo
    [34m2019-12-08 06:28:24.563 +07 [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004[0m Loaded configuration: /home/ratthapon/Blockchain/fabric-samples/first-network/configtx.yaml
    [34m2019-12-08 06:28:24.563 +07 [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005[0m Generating anchor peer update
    [34m2019-12-08 06:28:24.563 +07 [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006[0m Writing anchor peer update
    + res=0
    + set +x
    
    


ตรวจสอบไฟล์ที่ script gen มาให้


```bash
tree ./crypto-config
```

    [01;34m./crypto-config[00m
    ├── [01;34mordererOrganizations[00m
    │   └── [01;34mexample.com[00m
    │       ├── [01;34mca[00m
    │       │   ├── ca.example.com-cert.pem
    │       │   └── efd46609da859bba58893c07498c90dee40e709118c39d8ae2261a9de9381a4e_sk
    │       ├── [01;34mmsp[00m
    │       │   ├── [01;34madmincerts[00m
    │       │   ├── [01;34mcacerts[00m
    │       │   │   └── ca.example.com-cert.pem
    │       │   ├── config.yaml
    │       │   └── [01;34mtlscacerts[00m
    │       │       └── tlsca.example.com-cert.pem
    │       ├── [01;34morderers[00m
    │       │   ├── [01;34morderer2.example.com[00m
    │       │   │   ├── [01;34mmsp[00m
    │       │   │   │   ├── [01;34madmincerts[00m
    │       │   │   │   ├── [01;34mcacerts[00m
    │       │   │   │   │   └── ca.example.com-cert.pem
    │       │   │   │   ├── config.yaml
    │       │   │   │   ├── [01;34mkeystore[00m
    │       │   │   │   │   └── 4eed5a7a6a431d2bb13ca5a19cf382ae4f15f85689d15a680ad002ba976dae09_sk
    │       │   │   │   ├── [01;34msigncerts[00m
    │       │   │   │   │   └── orderer2.example.com-cert.pem
    │       │   │   │   └── [01;34mtlscacerts[00m
    │       │   │   │       └── tlsca.example.com-cert.pem
    │       │   │   └── [01;34mtls[00m
    │       │   │       ├── ca.crt
    │       │   │       ├── server.crt
    │       │   │       └── server.key
    │       │   ├── [01;34morderer3.example.com[00m
    │       │   │   ├── [01;34mmsp[00m
    │       │   │   │   ├── [01;34madmincerts[00m
    │       │   │   │   ├── [01;34mcacerts[00m
    │       │   │   │   │   └── ca.example.com-cert.pem
    │       │   │   │   ├── config.yaml
    │       │   │   │   ├── [01;34mkeystore[00m
    │       │   │   │   │   └── 16b23f06204fb2025ce6f02ddb5272eb110661556a89d5bb36d75595a090aed6_sk
    │       │   │   │   ├── [01;34msigncerts[00m
    │       │   │   │   │   └── orderer3.example.com-cert.pem
    │       │   │   │   └── [01;34mtlscacerts[00m
    │       │   │   │       └── tlsca.example.com-cert.pem
    │       │   │   └── [01;34mtls[00m
    │       │   │       ├── ca.crt
    │       │   │       ├── server.crt
    │       │   │       └── server.key
    │       │   ├── [01;34morderer4.example.com[00m
    │       │   │   ├── [01;34mmsp[00m
    │       │   │   │   ├── [01;34madmincerts[00m
    │       │   │   │   ├── [01;34mcacerts[00m
    │       │   │   │   │   └── ca.example.com-cert.pem
    │       │   │   │   ├── config.yaml
    │       │   │   │   ├── [01;34mkeystore[00m
    │       │   │   │   │   └── 3993ff56806551dfbf75771fe74980099f885c03fe8ad4e695e1aaef3df68aa4_sk
    │       │   │   │   ├── [01;34msigncerts[00m
    │       │   │   │   │   └── orderer4.example.com-cert.pem
    │       │   │   │   └── [01;34mtlscacerts[00m
    │       │   │   │       └── tlsca.example.com-cert.pem
    │       │   │   └── [01;34mtls[00m
    │       │   │       ├── ca.crt
    │       │   │       ├── server.crt
    │       │   │       └── server.key
    │       │   ├── [01;34morderer5.example.com[00m
    │       │   │   ├── [01;34mmsp[00m
    │       │   │   │   ├── [01;34madmincerts[00m
    │       │   │   │   ├── [01;34mcacerts[00m
    │       │   │   │   │   └── ca.example.com-cert.pem
    │       │   │   │   ├── config.yaml
    │       │   │   │   ├── [01;34mkeystore[00m
    │       │   │   │   │   └── e69ff901e335ace8862b70ea2e778db2dd35960d584455a0ffd5a998e1a3f201_sk
    │       │   │   │   ├── [01;34msigncerts[00m
    │       │   │   │   │   └── orderer5.example.com-cert.pem
    │       │   │   │   └── [01;34mtlscacerts[00m
    │       │   │   │       └── tlsca.example.com-cert.pem
    │       │   │   └── [01;34mtls[00m
    │       │   │       ├── ca.crt
    │       │   │       ├── server.crt
    │       │   │       └── server.key
    │       │   └── [01;34morderer.example.com[00m
    │       │       ├── [01;34mmsp[00m
    │       │       │   ├── [01;34madmincerts[00m
    │       │       │   ├── [01;34mcacerts[00m
    │       │       │   │   └── ca.example.com-cert.pem
    │       │       │   ├── config.yaml
    │       │       │   ├── [01;34mkeystore[00m
    │       │       │   │   └── fb93de5fe2c7bf649f070d3afbb9986629651edff4801c30217bc46707fedb24_sk
    │       │       │   ├── [01;34msigncerts[00m
    │       │       │   │   └── orderer.example.com-cert.pem
    │       │       │   └── [01;34mtlscacerts[00m
    │       │       │       └── tlsca.example.com-cert.pem
    │       │       └── [01;34mtls[00m
    │       │           ├── ca.crt
    │       │           ├── server.crt
    │       │           └── server.key
    │       ├── [01;34mtlsca[00m
    │       │   ├── 018e3732a87d0eb2041a7be64f62b2a97852f806572d0abd4c0a7371602bf3bb_sk
    │       │   └── tlsca.example.com-cert.pem
    │       └── [01;34musers[00m
    │           └── [01;34mAdmin@example.com[00m
    │               ├── [01;34mmsp[00m
    │               │   ├── [01;34madmincerts[00m
    │               │   ├── [01;34mcacerts[00m
    │               │   │   └── ca.example.com-cert.pem
    │               │   ├── config.yaml
    │               │   ├── [01;34mkeystore[00m
    │               │   │   └── 3a76baffb6f735dc1d9ce4cebfbb91db867418ba91449a567a80168b057140f4_sk
    │               │   ├── [01;34msigncerts[00m
    │               │   │   └── Admin@example.com-cert.pem
    │               │   └── [01;34mtlscacerts[00m
    │               │       └── tlsca.example.com-cert.pem
    │               └── [01;34mtls[00m
    │                   ├── ca.crt
    │                   ├── client.crt
    │                   └── client.key
    └── [01;34mpeerOrganizations[00m
        ├── [01;34morg1.example.com[00m
        │   ├── [01;34mca[00m
        │   │   ├── 0719a38de9c9b29074c6aa9b38d1287db097cc577a5ea2fdc80974e591363a45_sk
        │   │   └── ca.org1.example.com-cert.pem
        │   ├── [01;34mmsp[00m
        │   │   ├── [01;34madmincerts[00m
        │   │   ├── [01;34mcacerts[00m
        │   │   │   └── ca.org1.example.com-cert.pem
        │   │   ├── config.yaml
        │   │   └── [01;34mtlscacerts[00m
        │   │       └── tlsca.org1.example.com-cert.pem
        │   ├── [01;34mpeers[00m
        │   │   ├── [01;34mpeer0.org1.example.com[00m
        │   │   │   ├── [01;34mmsp[00m
        │   │   │   │   ├── [01;34madmincerts[00m
        │   │   │   │   ├── [01;34mcacerts[00m
        │   │   │   │   │   └── ca.org1.example.com-cert.pem
        │   │   │   │   ├── config.yaml
        │   │   │   │   ├── [01;34mkeystore[00m
        │   │   │   │   │   └── 6513ba9f376c419ec8f9b0163f925cf6b0aa9132539b082dc1c75939fede5c07_sk
        │   │   │   │   ├── [01;34msigncerts[00m
        │   │   │   │   │   └── peer0.org1.example.com-cert.pem
        │   │   │   │   └── [01;34mtlscacerts[00m
        │   │   │   │       └── tlsca.org1.example.com-cert.pem
        │   │   │   └── [01;34mtls[00m
        │   │   │       ├── ca.crt
        │   │   │       ├── server.crt
        │   │   │       └── server.key
        │   │   └── [01;34mpeer1.org1.example.com[00m
        │   │       ├── [01;34mmsp[00m
        │   │       │   ├── [01;34madmincerts[00m
        │   │       │   ├── [01;34mcacerts[00m
        │   │       │   │   └── ca.org1.example.com-cert.pem
        │   │       │   ├── config.yaml
        │   │       │   ├── [01;34mkeystore[00m
        │   │       │   │   └── 492a6d4296d4ca271302947e56b06d859def90f5a3f04244dbaa2e4f92bea601_sk
        │   │       │   ├── [01;34msigncerts[00m
        │   │       │   │   └── peer1.org1.example.com-cert.pem
        │   │       │   └── [01;34mtlscacerts[00m
        │   │       │       └── tlsca.org1.example.com-cert.pem
        │   │       └── [01;34mtls[00m
        │   │           ├── ca.crt
        │   │           ├── server.crt
        │   │           └── server.key
        │   ├── [01;34mtlsca[00m
        │   │   ├── e750db56e54a843b39ed4356eb4152a91d7ba8bad65c11cea7734cd68fb57877_sk
        │   │   └── tlsca.org1.example.com-cert.pem
        │   └── [01;34musers[00m
        │       ├── [01;34mAdmin@org1.example.com[00m
        │       │   ├── [01;34mmsp[00m
        │       │   │   ├── [01;34madmincerts[00m
        │       │   │   ├── [01;34mcacerts[00m
        │       │   │   │   └── ca.org1.example.com-cert.pem
        │       │   │   ├── config.yaml
        │       │   │   ├── [01;34mkeystore[00m
        │       │   │   │   └── 9d0a7eb14258b2f95b58d2bb944cee2d7618335915920a4ab15df0fd898886d2_sk
        │       │   │   ├── [01;34msigncerts[00m
        │       │   │   │   └── Admin@org1.example.com-cert.pem
        │       │   │   └── [01;34mtlscacerts[00m
        │       │   │       └── tlsca.org1.example.com-cert.pem
        │       │   └── [01;34mtls[00m
        │       │       ├── ca.crt
        │       │       ├── server.crt
        │       │       └── server.key
        │       └── [01;34mUser1@org1.example.com[00m
        │           ├── [01;34mmsp[00m
        │           │   ├── [01;34madmincerts[00m
        │           │   ├── [01;34mcacerts[00m
        │           │   │   └── ca.org1.example.com-cert.pem
        │           │   ├── config.yaml
        │           │   ├── [01;34mkeystore[00m
        │           │   │   └── 96f73c47e4ce7f7f92d4690d126c5273fce1267381d85a64221f62dfc71cc663_sk
        │           │   ├── [01;34msigncerts[00m
        │           │   │   └── User1@org1.example.com-cert.pem
        │           │   └── [01;34mtlscacerts[00m
        │           │       └── tlsca.org1.example.com-cert.pem
        │           └── [01;34mtls[00m
        │               ├── ca.crt
        │               ├── client.crt
        │               └── client.key
        └── [01;34morg2.example.com[00m
            ├── [01;34mca[00m
            │   ├── 3e55511f96b4551b848abd1041d2bec8f9156985cadd8477e56ac5fe39486c5f_sk
            │   └── ca.org2.example.com-cert.pem
            ├── [01;34mmsp[00m
            │   ├── [01;34madmincerts[00m
            │   ├── [01;34mcacerts[00m
            │   │   └── ca.org2.example.com-cert.pem
            │   ├── config.yaml
            │   └── [01;34mtlscacerts[00m
            │       └── tlsca.org2.example.com-cert.pem
            ├── [01;34mpeers[00m
            │   ├── [01;34mpeer0.org2.example.com[00m
            │   │   ├── [01;34mmsp[00m
            │   │   │   ├── [01;34madmincerts[00m
            │   │   │   ├── [01;34mcacerts[00m
            │   │   │   │   └── ca.org2.example.com-cert.pem
            │   │   │   ├── config.yaml
            │   │   │   ├── [01;34mkeystore[00m
            │   │   │   │   └── a14747bd57d7bc1028eb7e960888f2df7bed2df4783fe5d95fd826f565c5e3c7_sk
            │   │   │   ├── [01;34msigncerts[00m
            │   │   │   │   └── peer0.org2.example.com-cert.pem
            │   │   │   └── [01;34mtlscacerts[00m
            │   │   │       └── tlsca.org2.example.com-cert.pem
            │   │   └── [01;34mtls[00m
            │   │       ├── ca.crt
            │   │       ├── server.crt
            │   │       └── server.key
            │   └── [01;34mpeer1.org2.example.com[00m
            │       ├── [01;34mmsp[00m
            │       │   ├── [01;34madmincerts[00m
            │       │   ├── [01;34mcacerts[00m
            │       │   │   └── ca.org2.example.com-cert.pem
            │       │   ├── config.yaml
            │       │   ├── [01;34mkeystore[00m
            │       │   │   └── 56dd6cab1e15450c2e9cecea2a81cc80c25a85831e9b5c470ce0a08831f870f9_sk
            │       │   ├── [01;34msigncerts[00m
            │       │   │   └── peer1.org2.example.com-cert.pem
            │       │   └── [01;34mtlscacerts[00m
            │       │       └── tlsca.org2.example.com-cert.pem
            │       └── [01;34mtls[00m
            │           ├── ca.crt
            │           ├── server.crt
            │           └── server.key
            ├── [01;34mtlsca[00m
            │   ├── 0c7d9d187d1623719d63d56a5f14979609b5bb3863a5dacd30e539e6fa006267_sk
            │   └── tlsca.org2.example.com-cert.pem
            └── [01;34musers[00m
                ├── [01;34mAdmin@org2.example.com[00m
                │   ├── [01;34mmsp[00m
                │   │   ├── [01;34madmincerts[00m
                │   │   ├── [01;34mcacerts[00m
                │   │   │   └── ca.org2.example.com-cert.pem
                │   │   ├── config.yaml
                │   │   ├── [01;34mkeystore[00m
                │   │   │   └── 177e26826f96a12aa1df5cc5ba9c541624f743ceb633bee5caa0645eef18e658_sk
                │   │   ├── [01;34msigncerts[00m
                │   │   │   └── Admin@org2.example.com-cert.pem
                │   │   └── [01;34mtlscacerts[00m
                │   │       └── tlsca.org2.example.com-cert.pem
                │   └── [01;34mtls[00m
                │       ├── ca.crt
                │       ├── server.crt
                │       └── server.key
                └── [01;34mUser1@org2.example.com[00m
                    ├── [01;34mmsp[00m
                    │   ├── [01;34madmincerts[00m
                    │   ├── [01;34mcacerts[00m
                    │   │   └── ca.org2.example.com-cert.pem
                    │   ├── config.yaml
                    │   ├── [01;34mkeystore[00m
                    │   │   └── 02b2fa900c031222f8976b1c29e74eb7245c77a75e537159a59c5a100a937298_sk
                    │   ├── [01;34msigncerts[00m
                    │   │   └── User1@org2.example.com-cert.pem
                    │   └── [01;34mtlscacerts[00m
                    │       └── tlsca.org2.example.com-cert.pem
                    └── [01;34mtls[00m
                        ├── ca.crt
                        ├── client.crt
                        └── client.key
    
    141 directories, 133 files
    



```bash
tree ./channel-artifacts
```

    [01;34m./channel-artifacts[00m
    ├── channel.tx
    ├── genesis.block
    ├── Org1MSPanchors.tx
    └── Org2MSPanchors.tx
    
    0 directories, 4 files
    


# Step 3 Deploy Hyperledger Fabric Network

clean docker volume & image ก่อนทำการ deploy service


```bash
docker kill $(docker ps -q -f name=example.com) $(docker ps -q -f name=cli)
docker system prune -f
docker volume prune -f
```

    "docker kill" requires at least 1 argument.
    See 'docker kill --help'.
    
    Usage:  docker kill [OPTIONS] CONTAINER [CONTAINER...]
    
    Kill one or more running containers
    Deleted Containers:
    a4e787a6315d05190b5e1668686d6115ef97a7de04fed759a22dbf90ee0853f4
    52b78c6a180933224ce8784f8e00641482856f54ffc3d65f1d9a453ecbefcaa2
    
    Total reclaimed space: 0B
    Total reclaimed space: 0B
    



```bash
echo "y" | 
bash ./byfn.sh up -c getstartfabric
```

    Starting for channel 'getstartfabric' with CLI timeout of '10' seconds and CLI delay of '3' seconds
    proceeding ...
    LOCAL_VERSION=1.4.3
    DOCKER_IMAGE_VERSION=1.4.4
    =================== WARNING ===================
      Local fabric binaries and docker images are  
      out of  sync. This may cause problems.       
    ===============================================
    Creating network "net_byfn" with the default driver
    Creating volume "net_peer0.org2.example.com" with default driver
    Creating volume "net_peer1.org2.example.com" with default driver
    Creating volume "net_peer1.org1.example.com" with default driver
    Creating volume "net_peer0.org1.example.com" with default driver
    Creating volume "net_orderer.example.com" with default driver
    
    
    
    
    
    Creating peer0.org2.example.com
    Creating peer0.org1.example.com
    Creating peer1.org1.example.com
    Creating peer1.org2.example.com
    Creating orderer.example.com
    
    Creating cli
    [1BCONTAINER ID        IMAGE                               COMMAND             CREATED                  STATUS                  PORTS                      NAMES
    ad2c153f1e84        hyperledger/fabric-tools:latest     "/bin/bash"         Less than a second ago   Up Less than a second                              cli
    8919ae3f074b        hyperledger/fabric-orderer:latest   "orderer"           4 seconds ago            Up 2 seconds            0.0.0.0:7050->7050/tcp     orderer.example.com
    d20fa0b5a00e        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago            Up 1 second             0.0.0.0:10051->10051/tcp   peer1.org2.example.com
    1b723b4a77be        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago            Up Less than a second   0.0.0.0:8051->8051/tcp     peer1.org1.example.com
    b4972da8635e        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago            Up 1 second             0.0.0.0:9051->9051/tcp     peer0.org2.example.com
    d815e047ff1f        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago            Up 2 seconds            0.0.0.0:7051->7051/tcp     peer0.org1.example.com
    
     ____    _____      _      ____    _____ 
    / ___|  |_   _|    / \    |  _ \  |_   _|
    \___ \    | |     / _ \   | |_) |   | |  
     ___) |   | |    / ___ \  |  _ <    | |  
    |____/    |_|   /_/   \_\ |_| \_\   |_|  
    
    Build your first network (BYFN) end-to-end test
    
    Channel name : getstartfabric
    Creating channel...
    + peer channel create -o orderer.example.com:7050 -c getstartfabric -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    + res=0
    + set +x
    [34m2019-12-07 23:28:30.940 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:30.966 UTC [cli.common] readBlock -> INFO 002[0m Received block: 0
    ===================== Channel 'getstartfabric' created ===================== 
    
    Having all peers join the channel...
    + peer channel join -b getstartfabric.block
    + res=0
    + set +x
    [34m2019-12-07 23:28:31.012 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:31.065 UTC [channelCmd] executeJoin -> INFO 002[0m Successfully submitted proposal to join channel
    ===================== peer0.org1 joined channel 'getstartfabric' ===================== 
    
    + peer channel join -b getstartfabric.block
    + res=0
    + set +x
    [34m2019-12-07 23:28:34.112 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:34.202 UTC [channelCmd] executeJoin -> INFO 002[0m Successfully submitted proposal to join channel
    ===================== peer1.org1 joined channel 'getstartfabric' ===================== 
    
    + peer channel join -b getstartfabric.block
    + res=0
    + set +x
    [34m2019-12-07 23:28:37.252 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:37.316 UTC [channelCmd] executeJoin -> INFO 002[0m Successfully submitted proposal to join channel
    ===================== peer0.org2 joined channel 'getstartfabric' ===================== 
    
    + peer channel join -b getstartfabric.block
    + res=0
    + set +x
    [34m2019-12-07 23:28:40.395 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:40.488 UTC [channelCmd] executeJoin -> INFO 002[0m Successfully submitted proposal to join channel
    ===================== peer1.org2 joined channel 'getstartfabric' ===================== 
    
    Updating anchor peers for org1...
    + peer channel update -o orderer.example.com:7050 -c getstartfabric -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    + res=0
    + set +x
    [34m2019-12-07 23:28:43.584 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:43.591 UTC [channelCmd] update -> INFO 002[0m Successfully submitted channel update
    ===================== Anchor peers updated for org 'Org1MSP' on channel 'getstartfabric' ===================== 
    
    Updating anchor peers for org2...
    + peer channel update -o orderer.example.com:7050 -c getstartfabric -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    + res=0
    + set +x
    [34m2019-12-07 23:28:46.666 UTC [channelCmd] InitCmdFactory -> INFO 001[0m Endorser and orderer connections initialized
    [34m2019-12-07 23:28:46.673 UTC [channelCmd] update -> INFO 002[0m Successfully submitted channel update
    ===================== Anchor peers updated for org 'Org2MSP' on channel 'getstartfabric' ===================== 
    
    Installing chaincode on peer0.org1...
    + peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
    + res=0
    + set +x
    [34m2019-12-07 23:28:49.755 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001[0m Using default escc
    [34m2019-12-07 23:28:49.755 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002[0m Using default vscc
    [34m2019-12-07 23:28:49.946 UTC [chaincodeCmd] install -> INFO 003[0m Installed remotely response:<status:200 payload:"OK" > 
    ===================== Chaincode is installed on peer0.org1 ===================== 
    
    Install chaincode on peer0.org2...
    + peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
    + res=0
    + set +x
    [34m2019-12-07 23:28:50.000 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001[0m Using default escc
    [34m2019-12-07 23:28:50.000 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002[0m Using default vscc
    [34m2019-12-07 23:28:50.161 UTC [chaincodeCmd] install -> INFO 003[0m Installed remotely response:<status:200 payload:"OK" > 
    ===================== Chaincode is installed on peer0.org2 ===================== 
    
    Instantiating chaincode on peer0.org2...
    + peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C getstartfabric -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'AND ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
    + res=0
    + set +x
    [34m2019-12-07 23:28:50.211 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001[0m Using default escc
    [34m2019-12-07 23:28:50.211 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002[0m Using default vscc
    ===================== Chaincode is instantiated on peer0.org2 on channel 'getstartfabric' ===================== 
    
    Querying chaincode on peer0.org1...
    ===================== Querying on peer0.org1 on channel 'getstartfabric'... ===================== 
    + peer chaincode query -C getstartfabric -n mycc -c '{"Args":["query","a"]}'
    Attempting to Query peer0.org1 ...3 secs
    + res=0
    + set +x
    
    100
    ===================== Query successful on peer0.org1 on channel 'getstartfabric' ===================== 
    Sending invoke transaction on peer0.org1 peer0.org2...
    + peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C getstartfabric -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
    + res=0
    + set +x
    [34m2019-12-07 23:28:54.556 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001[0m Chaincode invoke successful. result: status:200 
    ===================== Invoke transaction successful on peer0.org1 peer0.org2 on channel 'getstartfabric' ===================== 
    
    Installing chaincode on peer1.org2...
    + peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
    + res=0
    + set +x
    [34m2019-12-07 23:28:54.604 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001[0m Using default escc
    [34m2019-12-07 23:28:54.604 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002[0m Using default vscc
    [34m2019-12-07 23:28:54.760 UTC [chaincodeCmd] install -> INFO 003[0m Installed remotely response:<status:200 payload:"OK" > 
    ===================== Chaincode is installed on peer1.org2 ===================== 
    
    Querying chaincode on peer1.org2...
    ===================== Querying on peer1.org2 on channel 'getstartfabric'... ===================== 
    Attempting to Query peer1.org2 ...3 secs
    + peer chaincode query -C getstartfabric -n mycc -c '{"Args":["query","a"]}'
    + res=0
    + set +x
    
    90
    ===================== Query successful on peer1.org2 on channel 'getstartfabric' ===================== 
    
    ========= All GOOD, BYFN execution completed =========== 
    
    
     _____   _   _   ____   
    | ____| | \ | | |  _ \  
    |  _|   |  \| | | | | | 
    | |___  | |\  | | |_| | 
    |_____| |_| \_| |____/  
    
    


ตรวจสอบการ deploy ว่าสำเร็จหรือไม่จาก log ที่ออกมา

ตรวจสอบ service ที่ถูก deploy ว่ามีอะไรบ้าง


```bash
docker ps
```

    CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS                  PORTS                      NAMES
    fbf88c3eb635        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   1 second ago        Up Less than a second                              dev-peer1.org2.example.com-mycc-1.0
    29a73802c5a4        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   5 seconds ago       Up 4 seconds                                       dev-peer0.org1.example.com-mycc-1.0
    3043228091dd        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   8 seconds ago       Up 7 seconds                                       dev-peer0.org2.example.com-mycc-1.0
    ad2c153f1e84        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              28 seconds ago      Up 27 seconds                                      cli
    8919ae3f074b        hyperledger/fabric-orderer:latest                                                                      "orderer"                32 seconds ago      Up 30 seconds           0.0.0.0:7050->7050/tcp     orderer.example.com
    d20fa0b5a00e        hyperledger/fabric-peer:latest                                                                         "peer node start"        32 seconds ago      Up 29 seconds           0.0.0.0:10051->10051/tcp   peer1.org2.example.com
    1b723b4a77be        hyperledger/fabric-peer:latest                                                                         "peer node start"        32 seconds ago      Up 28 seconds           0.0.0.0:8051->8051/tcp     peer1.org1.example.com
    b4972da8635e        hyperledger/fabric-peer:latest                                                                         "peer node start"        32 seconds ago      Up 29 seconds           0.0.0.0:9051->9051/tcp     peer0.org2.example.com
    d815e047ff1f        hyperledger/fabric-peer:latest                                                                         "peer node start"        32 seconds ago      Up 30 seconds           0.0.0.0:7051->7051/tcp     peer0.org1.example.com
    


# Step 4 Terminate Hyperledger Fabric Network

ทำการ shutdown และ clean docker container ต่างๆด้วย

*** ระวัง คำสั่งนี้จะทำการ kill docker container ทั้งหมดในเครื่อง ****


```bash
echo "y" |
bash ./byfn.sh down -c getstartfabric
```

    Stopping for channel 'getstartfabric' with CLI timeout of '10' seconds and CLI delay of '3' seconds
    proceeding ...
    [33mWARNING[0m: The BYFN_CA2_PRIVATE_KEY variable is not set. Defaulting to a blank string.
    [33mWARNING[0m: The BYFN_CA1_PRIVATE_KEY variable is not set. Defaulting to a blank string.
    
    
    
    
    
    
    
    
    
    
    
    
    [6BRemoving network net_byfn
    Removing volume net_peer0.org3.example.com
    [33mWARNING[0m: Volume net_peer0.org3.example.com not found.
    Removing volume net_peer1.org3.example.com
    [33mWARNING[0m: Volume net_peer1.org3.example.com not found.
    Removing volume net_orderer2.example.com
    [33mWARNING[0m: Volume net_orderer2.example.com not found.
    Removing volume net_orderer.example.com
    Removing volume net_peer0.org2.example.com
    Removing volume net_peer0.org1.example.com
    Removing volume net_peer1.org1.example.com
    Removing volume net_peer1.org2.example.com
    Removing volume net_orderer5.example.com
    [33mWARNING[0m: Volume net_orderer5.example.com not found.
    Removing volume net_orderer4.example.com
    [33mWARNING[0m: Volume net_orderer4.example.com not found.
    Removing volume net_orderer3.example.com
    [33mWARNING[0m: Volume net_orderer3.example.com not found.
    fbf88c3eb635
    29a73802c5a4
    3043228091dd
    Untagged: dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab:latest
    Deleted: sha256:71a46eb7b2062f663d835dba23b60e9c51b86933040dc9c06cb464e4e512da2c
    Deleted: sha256:60f8796515107af6a31d75774f847ea819ea8f2946fbfa5a5066d472b3382514
    Deleted: sha256:7ada5ddb7c1d18111ff31ded6e4722b8ae8b5381e96e78d76ef3b0d6e72f439f
    Deleted: sha256:189142f3c97d60a8ab8c532e86ed900f2abb7b2ed6e92e3f4076db7724c1b365
    Untagged: dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9:latest
    Deleted: sha256:55818851dbcfefee270703bb84c43aec593dddf21ac1652b7db960ec11e5ae54
    Deleted: sha256:03e520e22f30d293b26a918559aa928fa1940e7537cea772816b8c1e7593004d
    Deleted: sha256:7cdf4bdd9dd5b2cb58b032494477fa536973ba90ef3f659c54cb6dbac2d696eb
    Deleted: sha256:fc77da1f4e80fff0b2a2f3dd363e0617e3e7518572959b5181b5c75f38900c96
    Untagged: dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b:latest
    Deleted: sha256:7e50935e4242dc44ce5f66c57b5f106d9ad1d1ac7ba1f88cfaf445912b5e4813
    Deleted: sha256:56435f1e304f1b77663bcb04f0bef91020fa61d1907e5e887725cb28f0e87aa2
    Deleted: sha256:637811180f78062a86d2bad82eef6bd2a95fb3f5d6238dbcaed810796402cc3c
    Deleted: sha256:31fb6d39764dce95473116c3a8aca969ada4853dfb6ee1bd2a23499dbeb99e38
    





















































