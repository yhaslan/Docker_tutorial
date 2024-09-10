1. Create an anonymous volume
````
docker run -d -p 3000:80 --rm --name feedback-app -v /app/feedback
````

2. To create a named volume

````
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback
````

This one survives the container removal but anonymous volumes do not.

3. Bind Mounts
````
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app" -v /app/node_modules feedback-v
````

Bi tane anonymous volume yaratip npm install ile indirilen her seyi onun icine koyuyo sonra mount yapiyo ve boylece o volume syrvive ediyo ama ayni zamanda source code da da degisiklik yapilabiliyo

Just a quick note: If you don't always want to copy and use the full path, you can use these shortcuts:

macOS / Linux:
````
-v $(pwd):/app
````
Windows: 
````
-v "%cd%":/app
````

4. Read-only volumes
````
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app:ro" -v /app/node_modules feedback-v
````
adds ro to the end of bindmount command so that the change can only be made from source code to inside container and not vice versa

Yet the code above restricts our ability to write in temp and feedback folders but we wanna be able to write on them

Solution: actually -v feedback:/app/feedback since has a longer path it already overwrites the feedback folder, now we just need something similar for temp folder:
````
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-v
````

Anlamadigim feedbackinki anonymous degil, temp niye anonymous; himm session gecince unutulabilecek datalari mi koycaz diye

This ensures all volumes we need to write are rewritable and the rest are read

## Runtime Environment Variables
You set environment variables in Dockerfile

if your env var is port don't forget to change the listening port on server.js to process.env.PORT
they can be modified during the built, using -env or -e tags
````
docker run -d -p 3000:8000 -env PORT=8000 --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-v
````

or

````
docker run -d -p 3000:8000 -e PORT=8000 --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-v
````

or you can create a file .env as usual, and then:
````
docker run -d -p 3000:8000 --env-file ./.env --rm --name feedback-app -v feedback:/app/feedback -v "/home/yagmuraslan/Udemy_Docker/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-v
````

## Build Arguments
You can pass default values of certain vars in dockerfile and then build an initial image

And then use :
````
$docker build -t feedback:dev --build-arg DEFAULT_PORT=8000
````

this builds a second image, from the same base image and dockerifle without needing to change dockerfile each time. This second image is the same image but on different port