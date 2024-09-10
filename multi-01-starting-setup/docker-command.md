## Dockerizing mongodb
````
docker run --rm -d --name mongodb -p 27017:27017 mongo
````

## Dockerizing node app
First build a Dockerfile inside backend folder
````
backend$docker build -t goals-node .
backend$docker run --name goals-backend --rm goals_node
````
When you run this app it will crash a few seconds after start bcz cannot connect to the mongodb

Because the code in app.js made it accessible to localhost but now we r trying to access from inside a container. Again we'll need to change the localhost to host.docker.internal and rebuild our goals-node image

Then our backend container will connect to mongodb

But now the problem is the frontend cannot talk to the backend, because it talks through localhost IP address and backend no longer exposed to any port on local host. So modify again, remove the goals-backend container, and restart it but now with the correct port published:
````
backend$docker run --name goals-backend --rm -p 80:80 goals_node
````
you can also add -d

E bunu yapcaz da sonra react'i da containerize edince gene ayni sikinti cikmiycak mi? Orda da mi bi gidip host.docker.internal yapcaz

## Dockerizing React SPA
Go to frontend folder and create a Dockerfile . No official baseimage exists. This frontend setup in the end depends on Node. It is not a Node application but it uses Node to spin up the development server which serves this React application; and node is also used to optimize your code and transfer the code into a code browser understands

so base image is still Node

but you use npm start instead of node, app.js to tell it to use start script from package.json scripts

then build the image as usual
````
docker build -t goals-react .
docker run --rm --name goals-frontend -p 3000:3000 -d goals-react
````
But if you run this you will see the container automatically stops. this is something about React apps. You need to run them in interactive mode to inform that once you start and run you wanna keep entering commands
````
docker run --rm --name goals-frontend -p 3000:3000 -d -it goals-react
````

## Running containers on the same network 
so they wll talk to each other through the network and not through our host machine
````
docker network ls
````
to see available networks; but let's create a new one for this
````
docker network create goals-net
````
now let's build again the mongo container, this time no need to specify ports, they'll communicate thru same network
````
docker run --rm -d --name mongodb --network goals-net mongo
````
now continue with the backend
````
docker run --name goals-backend --rm -d --network goals-net goals-node
````
but before that we will have to change the image, the source code of backend; instead of host.docker.internal:27017 it should now talk to mongodb, as we did not publish the port this time. So change the source code and rebuild the image. cd backend i unutma

Finally React, first fix the code and then change the container run using network name; App.js folder da localhost gordugun her yere goals-backend yaz, cd frontend i unutma

docker run --rm --name goals-frontend --network goals-net -p 3000:3000 -d -it goals-react

#### bunda port u yine de yayinliycaksin ki localhostundan erisebilesin

### ERROR !
Sayfaya localhost:3000 den erisebiliyosun ama bi input girdigin an something went wrong!
This is because unlike node code, react code runs not in the container but in the browser. Therefore when we pass goals-backend instead of localhost browser has no idea what that is

This is not a bug but just exactly what react is. it helps us build apps running on the browser not on the server

Here the only thing running in the container is the server but the entire code inside App.js is running on the browser. So ?

We need to convert them back to localhost, bcz localhost is sth our browser can understand unlike goals-backend. AND THAT MEANS WE STILL NEED TO PUBLISH PORT 80 ON OUR BACKEND APPLICATION SO THAT THAT APP TOO BECOMES ACCESSIBLE ON LOCAL HOST;  

Ee ne anladik bu isten ?
There is no need to --network goal-net for frontend app because the development server its running on doesnt care about network; it is useful for backend container - mongodb communication but we should publish port 80 in backend container


#### Now working net we'll focus on Data persistance, limiting the access  and live source code update

To store mongoDB data externally, we will need to create a volume. but we don't know where inside mongodb container it is storing our data. we need to go to mongodb docker hub official image documentation page, it is telling us there

it's apparently under data/db. so this time when we start container we will do:
````
docker run --rm -d --name mongodb -v data:/data/db --network goals-net mongo
````
you can test if it works after entering a goal, stopping container -hence autoremoved- and re-running


Now, limit the access. Environment Varaibles under documentation
````
$ docker run -d --network some-network --name some-mongo \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo
````
````
docker run --rm -d --name mongodb -v data:/data/db --network goals-net -e MONGO_INITDB_ROOT_USERNAME=yagmur -e MONGO_INITDB_ROOT_PASSWORD=secret mongo

````
Now as you see when you refresh the fetching data fails and it fails because your node application connects without user credentials. Now you need to ensure that in app.js in backend you need to specify username and password for starting the database. how to do it is shown on mongo db documentation

it should be mongodb://username:password@host

therefore 'mongodb://yagmur:secret@mongodb:27017/course-goals?authSource=admin' sonuna authSource=admin eklemek zorundasin

IT still didnt work and apparently some other people ahd the same issue, so thz solution is; stop the mongodb container, remove the volume, restart the mongodb container, and then it should connect

#### Now we persisted the data and added access limitation for mongodb; next let's add data persistance for backend (ie log files) and live source code update for backend
````
odcker run --name goals-backend --rm -p 80:80 --network goals-net -v "/home/yagmuraslan/Udemy_Docker/multi-01-starting-setup/backend:/app" -v /app/node_modules -v logs:/app
/logs goals-node
````

but also you will need to add devDependencies start script and change CMD to npm, start

then you can do live updates in the source code

BUT is there an equivalent of nodemon for python apps ?

#### let's do some polishing; move username password to an environment variable
Ay bu kisim bana hic mantikli gelmedi

gene elimizle -e flag kullanarak giriyoruz ve run sirasidna degistiriyoruz


#### also don't forget dockerignore file



### Finally live source code updates for react app 
This time i am not binding the entire frontend folder to entire app folder in container

But i am only binding the src folders in each

This doesnt require any extra dependency the package.json already configured in a way that server will restart everytime there is a code change, the project setup already uses a wather like that
````
docker run --rm --name goals-frontend -p 3000:3000 -d -it -v "/home/yagmuraslan/Udemy_Docker/multi-01-starting-setup/frontend/src:/app/src" goals-react

````

simdi mesela git coursegoals.js de degisiklik yap, yansiycak
