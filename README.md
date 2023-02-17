# todo-list-app

A simple application to create Todo List to make _Getting Started tutorial_ of **docker labs**.

Application in NodeJS e MySQL.


## Create and run containers

### Build docker image
```
$ docker build -t todo-list-app .
```

### InMemory Database (SQLite)
```
$ docker run -dp 3000:3000 todo-list-app
```

### Using Mysql
First, create network
```
$ docker network create todo-app
```
Now, create a container with Mysql service.

```
$ docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:8.0
```

Checking logs of container:
```
$ docker exec -it <mysql-container-id> mysql -p todos
mysql> select * from todo_items;
+--------------------------------------+--------------------+-----------+
| id                                   | name               | completed |
+--------------------------------------+--------------------+-----------+
| c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
| 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
+--------------------------------------+--------------------+-----------+
```


### Application
Create a container of application and run it.
```
$ docker build -t todo-list-app .
$ docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```

Checking logs of container:
```
$ docker logs -f <container-id>
    # Previous log messages omitted
    $ nodemon src/index.js
    [nodemon] 2.0.20
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching path(s): *.*
    [nodemon] watching extensions: js,mjs,json
    [nodemon] starting `node src/index.js`
    Connected to mysql db at host mysql
    Listening on port 3000
```

### Using Docker Compose
Create docker-compose.yml file
```
services:
  app: 
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports: 
      - 3000:3000
    working_dir: /app
    volumes: 
      - ./:/app
    environment: 
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

Start up the application stack using the docker compose up command. 
We'll add the -d flag to run everything in the background.
```
docker compose up -d
```

Look at the logs using:
```
docker compose logs -f
```
The -f flag "follows" the log, so will give you live output as it's generated.

If you want to view the logs for a specific service, you can add the service name to the end of the logs command _(for example, `docker compose logs -f app`)_.

When you're ready to tear it all down, simply run:
```
$ docker compose down

# if you want remove volumes together, add --volumes flag:
$ docker compose down --volumes
```

### Caching build
We need to restructure our Dockerfile to help support the caching of the dependencies.

Change Dockerfile
```
FROM node:18-alpine
WORKDIR /app
+ COPY package.json yarn.lock ./
+ RUN yarn install --production
COPY . .
CMD ["node", "src/index.js"]
```
Add folder `node_modules` to .dockerignore.

Build image again.