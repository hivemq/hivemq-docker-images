# Running the image

## Basic single instance

To start a single HiveMQ Community Edition instance and allow access to the MQTT port as well as the Web UI, 
[get Docker](https://www.docker.com/get-started) and run the following command:

`docker run -p 8080:8080 -p 1883:1883 hivemq/hivemq-ce`

You can connect to the broker (1883) or the WebUI (8080) via the respective ports.