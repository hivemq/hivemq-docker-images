# Running the image

## Basic single instance

To start a single HiveMQ Community Edition instance and allow access to the MQTT port as well as the Web UI, 
[get Docker](https://www.docker.com/get-started) and run the following command:

`docker run -p 1883:1883 skobow/hivemq-ce`

You can connect to the broker via port 1883.
