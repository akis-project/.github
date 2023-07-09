# How To Run On Your Local? 💻 
## To run akis-flask-api 👇 
Create new environment **'venv'**
`python -m venv venv`

Activate **'venv'**
`source venv/Scripts/activate`

Pull the source code of **CTCWordBeamSearch**, an external dependency, from github
`git clone git@github.com:githubharald/CTCWordBeamSearch.git ./CTCWordBeamSearch`

Install dependencies
```
pip install -r requirements.txt
pip install ./CTCWordBeamSearch
```

Start the flask server
`flask --app server --debug run`

## To run akis-nodejs-api 👇
For development
`npm run dev`

For production
`npm start`

## To run akis-react-app 👇
`npm start`


# Ports Exposed
**akis-react-app:** `3000`
**akis-flask-api:** `5000`
**akis-nodejs-api:** `5001`


# Building Docker Images 📦 
After changing working directory to each project's root folder one by one, do the followings:

**akis-react-app**
`docker build . -t akisproject2023/akis-react-app:$(git branch --show-current)-$(git rev-parse --short HEAD)`

**akis-flask-api**
`docker build . -t akisproject2023/akis-flask-api:$(git branch --show-current)-$(git rev-parse --short HEAD)`

**akis-nodejs-api** 
`docker build . -t akisproject2023/akis-nodejs-api:$(git branch --show-current)-$(git rev-parse --short HEAD)`


# Pushing Docker Images to DockerHub 🚚 
**akis-react-app**
`docker push akisproject2023/akis-react-app:<tag here>`

**akis-flask-api**
`docker push akisproject2023/akis-flask-api:<tag here>`

**akis-nodejs-api**
`docker push akisproject2023/akis-nodejs-api:<tag here>`

# Running Docker Images 🚀 
**akis-react-app**
`docker run  --name akis-react-app -p 3000:3000 -d akisproject2023/akis-react-app:<tag here>`

**akis-flask-api**
`docker run  --name akis-flask-api -p 5000:5000 -d akisproject2023/akis-flask-api:<tag here>`

**akis-nodejs-api**
`docker run  --name akis-nodejs-api -p 5001:5001  -d akisproject2023/akis-nodejs-api:<tag here>`

