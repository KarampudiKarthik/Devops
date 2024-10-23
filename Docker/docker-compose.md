## Docker Compose

it is an declarative file

example: we will write the following command in yaml file

docker run -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb -d mongo

```
version: '3'
services:
	mongodb:         # container name
		image: mongo
		ports:
			- "27017:27017"
		environment:
			- MONGO_INITDB_ROOT_USERNAME=admin
			- MONGO_INITDB_ROOT_PASSWORD=password
```

Example: multi 
```
version: '3'
services:
	mongodb:         # container name
		image: mongo
		ports:
			- "27017:27017"
		environment:
			- MONGO_INITDB_ROOT_USERNAME=admin
			- MONGO_INITDB_ROOT_PASSWORD=password

	mongo-express:
		image: mongo-express
		restart: always
		ports:
			-"8081:8081"
		environment: 
			- ME_CONFIG_MONGODB_ADMINUSERNAME=admin
			- ME_CONFIG_MONGODB_ADMINPASSWORD=password
			- ME_CONFIG_MONGODB_SERVER=mongodb      # container name is the server
```
Example: with volumes
```
version: '3'
services:
	mongodb:         # container name
		image: mongo
		ports:
			- "27017:27017"
		environment:
			- MONGO_INITDB_ROOT_USERNAME=admin
			- MONGO_INITDB_ROOT_PASSWORD=password
		volumes:
			- mymongo-data:/data/db

	mongo-express:
		image: mongo-express
		restart: always
		ports:
			-"8081:8081"
		environment: 
			- ME_CONFIG_MONGODB_ADMINUSERNAME=admin
			- ME_CONFIG_MONGODB_ADMINPASSWORD=password
			- ME_CONFIG_MONGODB_SERVER=mongodb      # container name is the server

volumes:
	mymongo-data:
		driver: local
```















