# 13. Performance Testing guidance

Here some rules we get by experience when running performance testings:

1. Make sure the client can reach the desired throughput:
     - don't expect to get 20Gbps throughput from a single client having only 1 GbE NIC and 1 vCPU)
     - 
2. S3 Vendors cannot handle high TLS throughput or at least with strong encryption
     - show our PQC capability: PQC in transit (F5) completes PQC at rest (S3 partner)
     - lower TLS encryption on the backend
       
3. each S3 vendor come with their own CLI performance test tool (warp for minio, mongoose for Dell...), in priority use theirs.
     - in our lab we use **warp** provided by MinIO but you can use any other.
     
4. define a progressive detailed plan with different steps for exemple: 
    - 1 thread / 1M object
    - 10 threads / 1MB object
    - 100 threads / 1MB object
    - 1 thread / 10MB object
    - 1 thread / 100MB
    - 1 thread / 1GB
    - 10 threads / 100MB
    - 10 threads / 1GB
    ...
    so you can easily identify and trace potential issues (TLS, RPS, Throughput...)

5. Dashboard the results and highlight both the server side and the client side.
     -

Let's try it!
here it is a lab with limited resources so if we were to run too many clients with too big files we will end up with a message stating that the node volume is full

On the client, go on the **warp** directory
```bash
cd ~/warp
```

Then run a warp performance test tool against the VIP

```bash
./warp mixed --host=vip2.s3.f5demo.com:443 --tls --insecure --access-key="admin" --secret-key="myPass12345" --concurrent=10 --obj.size=1M --duration=15m --bucket=bucket1 --debug
```

here, we are running a 15 minutes test with 10 concurrent users with object sizes of 1MB. The test is **mixed** so it will measure uploads, downloads in single objects and multipart

Same as for the mc client, you can append a **debug** attribute if you want to look into requests details.
```bash
./warp mixed --host=vip2.s3.f5demo.com:443 --tls --insecure --access-key="admin" --secret-key="myPass12345" --concurrent=10 --obj.size=1M --duration=15m --bucket=bucket1 --debug
```

<p align="center">
	<img width="900" src="https://github.com/fchmainy/s3_lab/blob/main/docs/warp_benchmark_details.jpg?raw=true" alt="WARP Benchmark Explanations">
</p>
<br>
<br>
<br>

<br><br>
[Back to Agenda](/README.md)
