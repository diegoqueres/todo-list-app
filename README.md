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
$ docker logs <container-id>
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