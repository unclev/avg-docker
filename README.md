# Quick and "dirty" approach to run avgd (AVG Antivirus) in a docker container

The solution is inspired by [malice/avg](https://github.com/maliceio/malice-av/tree/master/avg). For whatever reason it didn't work to me.

This one can be used for occasional scanning a particular (pre-selected) location.

## Building
```bash
docker build -t unclev/avg .
```

## Run avg in a background
__It does not makes sense running it without the location mapped for testing__.
```bash
docker run --name avg_avgd_1 -d -v $(pwd):/malware unclev/avg
```

## Updating AVG in the running container
```bash
docker exec avg_avgd_1 avgupdate
```

## Scanning for malware
docker exec avg_avgd_1 avgscan .

See also [A real example](#a-real-example).

## Examples

### Starting in background (no mapping)
```bash
docker run --name avg_avgd_1 -d unclev/avg
```
the `docker logs` so far:
```
docker logs avg_avgd_1
 * Starting avgd ... (recovering)   ...done.
Checking for service avgd: (pid 27) is running
```
`ps aux | grep avg`:
```
root     12378  0.0  0.0  18044  2884 ?        Ss   16:38   0:00 /bin/bash -e /bin/avg
root     12426  5.5  0.7 334728 235128 ?       Sl   16:38   0:04 /opt/avg/av/bin//avgd
root     12875  5.5  0.0  71360  4936 ?        Sl   16:39   0:04 /opt/avg/av/bin/avgavid
root     12950  0.0  0.0 113112  5384 ?        Sl   16:39   0:00 /opt/avg/av/bin/avgtcpd
root     12956  0.0  0.0 330028 12816 ?        Sl   16:39   0:00 /opt/avg/av/bin/avgscand -c 3
root     12999  0.0  0.0 172304  5832 ?        Sl   16:39   0:00 /opt/avg/av/bin/avgsched
```

### Statistics
```
docker exec avg_avgd_1 avgctl --stat-all
AVG command line controller
Copyright (c) 2013 AVG Technologies CZ


------ AVG status ------
AVG version     :  13.0.3118
Components version :  Aspam:3111, Cfg:3109, Cli:3115, Common:3110, Core:4477, Doc:3115, Ems:3111, Initd:3113, Lng:3112, Oad:3118, Other:3109, Scan:3115, Sched:3110, Update:3109

Last update        :  Mon, 16 Jan 2017 10:31:08 +0000

------ License status ------
License number     :  LUOTY-674PL-VRWOV-APYEG-ZXHMA-E
License version    :  10
License type       :  FREE
License expires on :
Registered user    :
Registered company :

------ WD status ------
Component       State           Restarts        UpTime
Avid            running         0               3 minute(s)
Oad             error           0               -
Sched           running         0               3 minute(s)
Tcpd            running         0               3 minute(s)
Update          stopped         0               -


------ Sched status ------
Task name          Next runtime                      Last runtime
Virus update       -                                 -
Program update     -                                 -
User counting      Tue, 17 Jan 2017 10:42:13 +0000   Mon, 16 Jan 2017 10:42:13 +0000



------ Tcpd status ------
E-mails checked   :  0
SPAM messages     :  0
Phishing messages :  0
E-mails infected  :  0
E-mails dropped   :  0


------ Avid status ------
Virus database reload times     :  0
Virus database version          :  4477/13780
Virus database release date     :  Mon, 16 Jan 2017 06:31:00 +0000
Virus database shared in memory :  yes

Operation successful.
```

### List */malvare* location
```
docker exec avg_avgd_1 ls -Al /malware
total 4
-rw------- 1 root root 68 Jan  1  1970 EICAR
```

### Scanning for malware
```
docker exec avg_avgd_1 avgscan .
AVG command line Anti-Virus scanner
Copyright (c) 2013 AVG Technologies CZ

Virus database version: 4477/13780
Virus database release date: Mon, 16 Jan 2017 06:31:00 +0000

./EICAR  Virus identified EICAR_Test

Files scanned     :  1(1)
Infections found  :  1(1)
PUPs found        :  0
Files healed      :  0
Warnings reported :  0
Errors reported   :  0
```

### Updating AVG in a running container
```
docker exec avg_avgd_1 avgupdate
AVG command line update
Copyright (c) 2013 AVG Technologies CZ

Running update.
Initializing...
Analyzing...
100% [===================================>]

You are currently up-to-date
Update was successfully completed.
```

### Stop the AVG container and script output
```
docker stop avg_avgd_1 ; docker logs avg_avgd_1
avg_avgd_1
 * Starting avgd ... (recovering)   ...done.
Checking for service avgd: (pid 27) is running
Checking for service avgd: (pid 27) is running
 * Stopping avgd.   ...done.
Checking for service avgd: avgd is not running
```

### Start the stopped container
```
docker stop avg_avgd_1 ; docker logs avg_avgd_1
avg_avgd_1
 * Starting avgd ... (recovering)   ...done.
Checking for service avgd: (pid 27) is running
Checking for service avgd: (pid 27) is running
 * Stopping avgd.   ...done.
Checking for service avgd: avgd is not running
 * Starting avgd ... (recovering)   ...done.
Checking for service avgd: (pid 23) is running
Checking for service avgd: (pid 23) is running
 * Stopping avgd.   ...done.
Checking for service avgd: avgd is not running
```
Actually it started docker daemon once.

### Stop the container and remove it
```bash
docker stop avg_avgd_1 && docker rm avg_avgd_1
```

### A real example
```
$ ls -Al
итого 948
drwxr-xr-x 3 victor victor   4096 сен  7  2015 Program Files
-rw-r--r-- 1 victor victor 484864 июл  9  2015 QRCoderSetup.msi
-rwxr-xr-x 1 victor victor 476168 янв  9 10:10 setup.exe
docker run --name avg_avgd_2 -v $(pwd):/malware unclev/avg avgscan .
 * Starting avgd ... (recovering)   ...done.
Checking for service avgd: (pid 27) is running
AVG command line Anti-Virus scanner
Copyright (c) 2013 AVG Technologies CZ

Virus database version: 4477/13780
Virus database release date: Mon, 16 Jan 2017 06:31:00 +0000


Files scanned     :  12(4)
Infections found  :  0(0)
PUPs found        :  0
Files healed      :  0
Warnings reported :  0
Errors reported   :  0

Checking for service avgd: (pid 27) is running
 * Stopping avgd.   ...done.
 ```
