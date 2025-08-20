FROM node:24.5.0-alpine3.22
WORKDIR /app
COPY . .
RUN npm install && \
	npm audit fix
EXPOSE 4000
CMD npm start
