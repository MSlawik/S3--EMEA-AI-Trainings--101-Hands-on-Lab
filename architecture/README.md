## 2. Lab Architecture

### 2.1 Diagram

<p align="center">
	<img width="800" src="https://github.com/fchmainy/s3_lab/blob/main/docs/architecture_s3_udf_lab.jpg?raw=true" alt="UDF S3 Topo">
</p>
<br>
<br>
<br>
### 2.2 Lab Components

| Virtual Machine | OS | Management | External IP Addresses | Internal IP Address |
|--|--|-|--|--|
| Client | Ubuntu 22.04 | 10.1.1.101 | 10.1.10.101 | None |
| BIG-IP | TMOS v21.0 | 10.1.1.11 | 10.1.10.11 | 10.1.20.11 |
| S3 Server | Ubuntu 22.04 | 10.1.1.51 | None | 10.1.20.101 (minio-101) {"Console": "TCP/9001", "API": "TCP/9000"}  |
|||||10.1.20.102 (minio-102) |
|||||10.1.20.103 (minio-103) |
|||||10.1.20.104 (minio-104) |
|||||10.1.20.105 (minio-105) |
| Analytics | Ubuntu 22.04 | 10.1.1.61 | None | 10.1.20.201 |
|--|--|--|--|--|

<br>
<br>
### 2.3 Lab details you need to be aware of

This lab has been designed to work with Minio Open Source. 
Very recently (Dec 2025), MinIO has put their OpenSource version under maintenance mode and release AIStor Free in replacement. We need to update the lab using this.

In the lab, MinIO nodes are running as standalone docker containers with their own volumes on the same Ubuntu VM to keep a fair balance between a good design and a lab price. This is easier and cheaper to run but it is also a bit longer to configure (each administrative configurations has to be executed on all nodes) - take it as repetitive tasks so you remember them :)

I really like MinIO, first because they are great partners of F5 and second because they are very close to AWS S3 specifications.


<br><br>
[Back to Agenda](/README.md)
