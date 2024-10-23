Example: Basic flask app.

task:
run this on 3000 port

```
FROM python:3-alpine3.15
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 3000
CMD python ./index.py  # run this file
```

build  the image 
* first we need to login
```
docker build -t krazy666777/hey-python-flask:0.1.RELEASE .
```
to run container
```
docker container run -d -p 3000:3000 krazy666777/hey-python-flask:0.1.RELEASE
```
