# Description

This project consists of several components, each being kept in their own repository.
These components are:

1. [akis-react-app](https://github.com/akis-project/akis-react-app)
2. [akis-nodejs-api](https://github.com/akis-project/akis-nodejs-api)
3. [akis-flask-api](https://github.com/akis-project/akis-flask-api)
4. [docker-compose](https://github.com/akis-project/docker-compose)
5. akis-mongo
6. **[akis-proxy ](https://github.com/akis-project/akis-proxy)(only in production environment)**

## General structure

The head component is

- **`docker-compose`** 🧠
  - Stores config files
  - Manages services
  - Manages volumes
  - Manages networks
  - Manages environment variables

There are two versions of it:

- `docker-compose.prod.yml` for production
- `docker-compose.dev.yml` for local development
  Since we don't have a different standalone instance for development, we use `dev` environment for local development purposes.

Both files will look as follows
![image](https://github.com/akis-project/.github/assets/68128434/22d16cda-8f1d-41b5-9a14-3fa0bcfaa177)
Notice the difference that `prod` environment has an extra `akis-proxy` service.
It is not needed in local development ⚠️

### Service definitions

- **`akis-react-app`**
  - Serves UI
  - Communicates with akis-nodejs-api
  - Runs on port 3000 (dev)
  - Runs on port 3001 (prod)
- **`akis-nodejs-api`**
  - Runs express server
  - Runs backend logic
  - Communicated with `akis-react-app`, `akis-flask-api`, and `mongo`
  - Hosts images statically to `akis-react-app`
  - Shares volume between `akis-flask-api`
  - Runs on port 5001 (always)
- **`akis-flask-api`**
  - Runs flask server
  - Runs machine learning model
  - Gets image from `akis-nodejs-api` and returns transcription results back.
  - Saves images and crops to local directory of host instance
  - Shares volume with `akis-nodejs-api`
  - Runs on port 5000 (always)
- **`mongo`**
  - Runs NoSQL database
  - Stores user information, transcription results and sessions.
  - Runs on port 27017
- **`akis-proxy`** (only in prod ⚠️)
  - Proxy layer between `demos.sabanciuniv.edu` and host instance.
  - Utilizes [node-http-proxy](https://www.npmjs.com/package/node-http-proxy) library
  - Receives requests from `demos.sabanciuniv.edu`
  - Forwards requests to host instance's `3001 `(react) and `5001 `(nodejs) ports
  - Runs on port 3000 (prod)
  - Why do we need such proxying?
    - All the docker containers are designed to communicate each other using internal docker network.
    - `akis-react-app` is the frontend service, it will store session cookies on the browser
    - `akis-nodejs-api` is the backend service which stores session, i.e. it is cookie-setter of the frontend
    - And there is a reverse-proxy set in `demos.sabanciuniv.edu`'s machine which forwards HTTPS(443) requests to `vpa-com16`'s 3000 port.
    - But by its nature, backend service is designed to set cookies in client
    - So, because of this reverse-proxying of `demos.sabanciuniv.edu`, the client should be demos.sabanciuniv.edu
    - And `akis-proxy` service will handle this request forwarding among `react `and `nodejs `.
    - So that backend will assume that incoming request is actually coming from `demos` even though it is designed to receive requests from vpa-com16's 3000 port. Then it will set cookies in `demos` successfully ✅ .
    - If we didn't have this `akis-proxy` service, users who visit demos wouldn't be able to sign-in even though their credentials were true. This is because of [cross-origin cookies](https://medium.com/@sharadokey/understanding-cors-and-cross-origin-cookies-bf36d624da78).
    - Also one more benefit of `akis-proxy` is it hides other `vpa-com16'`s services/containers from `demos`. `demos` will only know about `vpa-com16`'s 3000 port and it won't talk to any other services/containers of `vpa-com16`.

### Volume definitions

There are two local bind volumes:

```yml
db:
  driver: local
  driver_opts:
    type: "none"
    o: "bind"
    device: "../volumes/db"

uploads:
  driver: local
  driver_opts:
    type: "none"
    o: "bind"
    device: "../volumes/uploads"
```

- **`db`**
  - MongoDB's volume
  - Persistent (will remain even if you terminate services/containers)
- **`uplaods`**
  - Shared between flask and nodejs services
  - flask will save images in this directory
  - nodejs will read images from this directory

### Network definitions

Nothing fancy. Networks between services should be set `internal: true`
If you want to connect to MongoDB instance from public internet, set `internal: false`

```yml
react-nodejs-network:
  driver: bridge
  ipam:
    driver: default
    config:
      - subnet: 172.16.1.0/28

nodejs-mongo-network:
  driver: bridge
  internal: false # False to allow access from internet
  ipam:
    driver: default
    config:
      - subnet: 172.16.2.0/29

nodejs-flask-network:
  driver: bridge
  internal: false # False to allow access from internet
  ipam:
    driver: default
    config:
      - subnet: 172.16.3.0/29
```

### Schemas

Service schema
![service schema](https://github.com/akis-project/.github/assets/68128434/0f4ffe76-ac5d-4865-bd72-948021d6950a)

Volume schema
![volume schema](https://github.com/akis-project/.github/assets/68128434/07a32837-f5d2-4582-8947-55ad47203725)

Proxying schema in production
![proxy](https://github.com/akis-project/.github/assets/68128434/b1fe8876-6518-4fc8-be76-a2d1aa61bf62)

Further details about each component is available in the documentation 📝

# How to run the project locally💻

### Prerequisites

Ensure that you have the followings installed before you attempt to run the project in your computer:
| [Docker ](https://www.docker.com/)| 24.0.4, build 3713ee1 |
|--------|-----------------------|

- Note that the reason why docker is used in local development is because of volume bindings.

## Steps

1. Create a folder named `akis-project` somewhere in your disk.
2. Clone repositories from github
   - [akis-react-app](https://github.com/akis-project/akis-react-app)
   - [akis-nodejs-api](https://github.com/akis-project/akis-nodejs-api)
   - [akis-flask-api](https://github.com/akis-project/akis-flask-api)
   - [docker-compose](https://github.com/akis-project/docker-compose)
3. Switch to respective branches of each repository:

```sh
# akis-react-app
git switch dev

# akis-nodejs-api
git switch dev

# akis-flask-api
git switch dev

# docker-compose
# stay in 'main'
```

4. Create volumes in project directory (you must be in akis-project folder, at the root)

> ⚠️Because we are binding local volumes to docker containers, we need to create those volumes manually in the host machine
> ⚠️This part is a MUST!
> ⚠️Otherwise you won't be able to run `docker compose up`

```sh
mkdir volumes
cd volumes
mkdir db
mkdir uploads
```

After this step, make sure you have the following folder structure:
![image](https://github.com/akis-project/.github/assets/68128434/7dff8b3f-41d1-42f7-8df6-52f644557494)

6. Change environment variables & run the docker compose

```sh
mv .env.dev .env
mv docker-compose.dev.yml docker-compose.yml

# To see logs:
docker compose up

# To run in detached mode:
# docker compose up -d
```

### Conclusion

- From now on, docker containers will be running on the background, with volumes and networks managed by itself automatically.
- `React` and `nodejs` containers will be running in development mode, i.e. whenever you make changes in the code, they will hot reload. You will be able to see the latest changes without restarting containers 👍
- **However, `flask` will need to be restarted again when its code is updated** 🛑
