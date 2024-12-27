My macOS setup is Apple M3 Max with 128 GB of RAM.

## Step 1. Clone the repository and navigate to the directory
```shell
git clone https://github.com/infiniflow/ragflow.git
cd ragflow
```
## Step 2. Configure the .env file
Edit the docker/.env file:
```shell
vi docker/.env
Configure as needed:
# Set the RAGFLOW_IMAGE to use a domestic (China) mirror if required.
# If you are using macOS, uncomment the line MACOS=1.
```
## Step 3. Start the services
```shell
docker compose -f docker/docker-compose.yml up -d
```
Note: The initial startup may take several minutes, depending on the image size, network speed, and host performance.
After finished, open http://127.0.0.1 in your browser and check if the RagFlow login page loads successfully.

## **Troubleshoot Issue 1**: Registration and login are unresponsive

Check the logs of the ragflow-server container:
```shell
docker logs -f ragflow-server
```
and found the following error:
```shell
The following required CPU features were not detected:
    avx, avx2, fma, bmi1, bmi2, lzcnt, movbe
Continuing to use this version of Polars on this processor will likely result in a crash.
Install the `polars-lts-cpu` package instead of `polars` to run Polars with better compatibility.
```
Solution: Enter the container and install polars-lts-cpu:
```shell
docker exec -it ragflow-server /bin/bash
pip install polars-lts-cpu
exit
docker restart ragflow-server
```
Retested registration and login functionality, and found it is now OK. 

## **Troubleshoot Issue 2**: Errors related to Elasticsearch (ES01) missing during chat

Occured the following error while chatting
```
elastic_transport.ConnectionError: Connection error caused by: ConnectionError(Connection error caused by: NameResolutionError(<urllib3.connection.HTTPConnection object at 0x7ffeb1b840d0>: Failed to resolve 'es01' ([Errno -2] Name or service not known)))
2024-12-27 17:36:09,188 INFO     27 172.18.0.6 - - [27/Dec/2024 17:36:09] "POST /v1/conversation/completion HTTP/1.1" 200 -
2024-12-27 17:36:23,906 INFO     28 task_consumer_0 reported heartbeat: {"name": "task_consumer_0", "now": "2024-12-27T17:36:23.903+08:00", "boot_at": "2024-12-27T17:27:57.772+08:00", "pending": 0, "lag": 0, "done": 1, "failed": 0, "current": null}
```
Check the status of the ragflow-es-01 container:
```shell
docker ps
docker logs -f ragflow-es-01
```
Identified the container is exiting due to insufficient memory (Exit Code 137):
Solution: Open Docker Desktop Settings，then increase the memory allocation to 16GB (at least 4GB minimum). After that, restart all RagFlow-related services.

Check the logs for both ragflow-server and ragflow-es-01 containers to ensure there are no errors:
```shell
docker logs -f ragflow-server
docker logs -f ragflow-es-01
```

Finally, everything is working fine.
