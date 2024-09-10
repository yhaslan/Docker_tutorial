1. Build mongo db image, inspect it to find the IP address of image and then pass that address to the app.js mongodb;
````
docker run -d -t mongodb mongo .

docker container inspect mongo db
````

then build the image and container for the favorites app


2. Network approach : you can create container networks in docker

When containers are on the same network they can talk to java (??°) each other diyo altyazi hatali
````
docker network create favorites-net

docker run -d --name mongodb --network favorites-net mongo .
````

If two containers are part of the same network, instead of handcoding the ip address you can put the name of the other container there

