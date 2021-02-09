## Updates on https://github.com/mburakunuvar/dockerized-fullstackapp

### Production Multi-Container Deployments

We'll take a different approach to prevent from 2nd build. Travis will push the images to DockerHub and Elastic Beanstalk will use those to run images, instead of building from scratch.

Amazon Web Services and Google Cloud and many other providers are setup out of the box to automatically download images from Dakar hub specifically without a whole bunch of additional configuration. And so we really considered Docker-Hub to be like a central repository for everything in the docker world.

### Production Dockerfiles

in order to replace all dockerfile.dev docs with prod version

- express-worker only change `CMD ["npm", "run", "start"]`
- express-server only change `CMD ["npm", "run", "start"]`
- nginx no change
- nginx remove `sockjs-node` from default.md as we no more need disk mapping

### Multiple Nginx Instances

So this time we're still going to have Nginx server that is solely dedicated to serving up our production re-act files. But this time rather than listening on port 80 and being like the first jump inside the elastic
beanstalk instance it's going to be listening on port 3000 and users are only going to be able access this Nginx server before - by all of our production files by first going through the other copy Nginx

note : So it just happens that in our example right now we are using Nginx for both of these servers. It could be very well possible that we are using different services for the two.

to sum up :

- nginx container 1 => routing across react and node
- nginx container 2 => serving react

#### Altering Nginx's Listen Port (not the router but the server ;)

- create a folder within react-frontend
- create a new default.conf for server nginx

```conf
server {
  listen 3000;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
}
```

- Production Dockerfiles for react-frontend ( multi step build)

```Dockerfile
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
# `/app/build` => all the stuff we care about
# plus, new config files for nginx
FROM nginx
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/build /usr/share/nginx/html
```

- Update App.test.js
  Clean up tests ( as there'll be no connection to backend yet)
  FIB component is going to try to make a request to our back and express server. That is not quite running at this point in time.

## Github and Travis CI Setup

## Pushing Images to Docker Hub
