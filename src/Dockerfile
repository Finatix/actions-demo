FROM node:alpine3.19

WORKDIR /usr/src/app
RUN chown -R node /usr/src/app
USER node
COPY --chown=node ["package.json", "package-lock.json*", "./"]
RUN npm install
COPY --chown=node . .
RUN npm install
RUN npm run build