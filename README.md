0x04. Files manager
Back-end
JavaScript
ES6
NoSQL
MongoDB
Redis
NodeJS
ExpressJS
Kue
 By: Guillaume, CTO at Holberton School
 Weight: 1
 Project to be done in teams of 2 people (your team: Isaiah Talabi)
 Project will start Nov 30, 2023 5:00 AM, must end by Dec 7, 2023 5:00 AM
 Checker was released at Dec 1, 2023 11:00 PM
 Manual QA review must be done (request it when you are done with the project)
 An auto review will be launched at the deadline
This project is a summary of this back-end trimester: authentication, NodeJS, MongoDB, Redis, pagination and background processing.

The objective is to build a simple platform to upload and view files:

User authentication via a token
List all files
Upload a new file
Change permission of a file
View a file
Generate thumbnails for images
You will be guided step by step for building it, but you have some freedoms of implementation, split in more files etc… (utils folder will be your friend)

Of course, this kind of service already exists in the real life - it’s a learning purpose to assemble each piece and build a full product.

Enjoy!

Resources
Read or watch:

Node JS getting started
Process API doc
Express getting started
Mocha documentation
Nodemon documentation
MongoDB
Bull
Image thumbnail
Mime-Types
Redis
Learning Objectives
At the end of this project, you are expected to be able to explain to anyone, without the help of Google:

how to create an API with Express
how to authenticate a user
how to store data in MongoDB
how to store temporary data in Redis
how to setup and use a background worker
Requirements
Allowed editors: vi, vim, emacs, Visual Studio Code
All your files will be interpreted/compiled on Ubuntu 18.04 LTS using node (version 12.x.x)
All your files should end with a new line
A README.md file, at the root of the folder of the project, is mandatory
Your code should use the js extension
Your code will be verified against lint using ESLint
Provided files
package.json
Click to show/hide file contents
.eslintrc.js
Click to show/hide file contents
babel.config.js
Click to show/hide file contents
and…
Don’t forget to run $ npm install when you have the package.json

Tasks
0. Redis utils
mandatory
Inside the folder utils, create a file redis.js that contains the class RedisClient.

RedisClient should have:

the constructor that creates a client to Redis:
any error of the redis client must be displayed in the console (you should use on('error') of the redis client)
a function isAlive that returns true when the connection to Redis is a success otherwise, false
an asynchronous function get that takes a string key as argument and returns the Redis value stored for this key
an asynchronous function set that takes a string key, a value and a duration in second as arguments to store it in Redis (with an expiration set by the duration argument)
an asynchronous function del that takes a string key as argument and remove the value in Redis for this key
After the class definition, create and export an instance of RedisClient called redisClient.

bob@dylan:~$ cat main.js
import redisClient from './utils/redis';

(async () => {
    console.log(redisClient.isAlive());
    console.log(await redisClient.get('myKey'));
    await redisClient.set('myKey', 12, 5);
    console.log(await redisClient.get('myKey'));

    setTimeout(async () => {
        console.log(await redisClient.get('myKey'));
    }, 1000*10)
})();

bob@dylan:~$ npm run dev main.js
true
null
12
null
bob@dylan:~$ 
Repo:

GitHub repository: alx-files_manager
File: utils/redis.js
   
1. MongoDB utils
mandatory
Inside the folder utils, create a file db.js that contains the class DBClient.

DBClient should have:

the constructor that creates a client to MongoDB:
host: from the environment variable DB_HOST or default: localhost
port: from the environment variable DB_PORT or default: 27017
database: from the environment variable DB_DATABASE or default: files_manager
a function isAlive that returns true when the connection to MongoDB is a success otherwise, false
an asynchronous function nbUsers that returns the number of documents in the collection users
an asynchronous function nbFiles that returns the number of documents in the collection files
After the class definition, create and export an instance of DBClient called dbClient.

bob@dylan:~$ cat main.js
import dbClient from './utils/db';

const waitConnection = () => {
    return new Promise((resolve, reject) => {
        let i = 0;
        const repeatFct = async () => {
            await setTimeout(() => {
                i += 1;
                if (i >= 10) {
                    reject()
                }
                else if(!dbClient.isAlive()) {
                    repeatFct()
                }
                else {
                    resolve()
                }
            }, 1000);
        };
        repeatFct();
    })
};

(async () => {
    console.log(dbClient.isAlive());
    await waitConnection();
    console.log(dbClient.isAlive());
    console.log(await dbClient.nbUsers());
    console.log(await dbClient.nbFiles());
})();

bob@dylan:~$ npm run dev main.js
false
true
4
30
bob@dylan:~$ 
Repo:

GitHub repository: alx-files_manager
File: utils/db.js
   
2. First API
mandatory
Inside server.js, create the Express server:

it should listen on the port set by the environment variable PORT or by default 5000
it should load all routes from the file routes/index.js
Inside the folder routes, create a file index.js that contains all endpoints of our API:

GET /status => AppController.getStatus
GET /stats => AppController.getStats
Inside the folder controllers, create a file AppController.js that contains the definition of the 2 endpoints:

GET /status should return if Redis is alive and if the DB is alive too by using the 2 utils created previously: { "redis": true, "db": true } with a status code 200
GET /stats should return the number of users and files in DB: { "users": 12, "files": 1231 } with a status code 200
users collection must be used for counting all users
files collection must be used for counting all files
Terminal 1:

bob@dylan:~$ npm run start-server
Server running on port 5000
...
Terminal 2:

bob@dylan:~$ curl 0.0.0.0:5000/status ; echo ""
{"redis":true,"db":true}
bob@dylan:~$ 
bob@dylan:~$ curl 0.0.0.0:5000/stats ; echo ""
{"users":4,"files":30}
bob@dylan:~$ 
Repo:

GitHub repository: alx-files_manager
File: server.js, routes/index.js, controllers/AppController.js
   
