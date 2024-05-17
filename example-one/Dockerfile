# start from a Node.js 21 image
FROM node:21

# specify the directory inside the image in which all commands will run
WORKDIR /usr/src/app

# Copy the package files and install dependencies
COPY package*.json ./
RUN npm install
RUN npm install express

# COPY all of the application files into the image
COPY . .

# Specify the default command to run when starting the container
CMD ["npm", "start"]