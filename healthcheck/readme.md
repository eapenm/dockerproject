# HealthCheck in docker files

### Example flask file
Let's start with the requirements.txt:
```
Flask==0.12.2
```
And the Dockerfile:

```
FROM python:3.6-alpine

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt

CMD ["python", "app.py"]

```

And finally, app.py:

```
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello world'


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

Now let's build the container:
```
docker build -t docker-flask .
```

This should build pretty quickly. Then we'll run the container.
```
docker run --rm --name docker-flask -p 5000:5000 docker-flask
```

Now test by opening up your browser to localhost:5000. You should see "Hello world".

### Add a health check to the Dockerfile

A health check is configured in the Dockerfile using the HEALTHCHECK instruction. There are two ways to use the HEALTHCHECK instruction:
```
HEALTHCHECK [OPTIONS] CMD command
```
or if you want to disable a health check from a parent image:
```
HEALTHCHECK NONE
```
So add this line to the Dockerfile right before the last line (CMD).
```
HEALTHCHECK CMD curl --fail http://localhost:5000/ || exit 1
```
like below:
```
FROM python:3.6-alpine

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt
HEALTHCHECK CMD curl --fail http://localhost:5000/ || exit 1
CMD ["python", "app.py"]
```
### See the health status

Let's rebuild and run our container.
```
docker build -t docker-flask .
docker run --rm --name docker-flask -p 5000:5000 docker-flask
```
Now let's take a look at the health status. Notice we have the --name option to the above command so we can easily inspect the container.
```
docker inspect --format='{{json .State.Health}}' docker-flask 
```
If you run this immediately after the container starts, you'll see Status is starting.
```
{"Status":"starting","FailingStreak":0,"Log":[]}
```
And after the health check runs (after the default interval of 30s):
```
{"Status":"healthy","FailingStreak":0,"Log":[{"Start":"2017-07-21T06:10:51.809087707Z","End":"2017-07-21T06:10:51.868940223Z","ExitCode":0,"Output":"Hello world"}]}
```
## Configure the health check using a compose file
We can also configure the health check using a compose file. Let's make a file called docker-compose.yml.
```
version: '3.1'

services:
  web:
    image: docker-flask
    ports:
      - '5000:5000'
```      
And we deploy our stack to a swarm using:

docker stack deploy --compose-file docker-compose.yml flask
Now, the health check was specified in the Dockerfile, but we can also specify (or override) the health check settings in our compose file.

```
version: '3.1'

services:
  web:
    image: docker-flask
    ports:
      - '5000:5000'
    healthcheck:
      test: curl --fail -s http://localhost:5000/ || exit 1
      interval: 1m30s
      timeout: 10s
      retries: 3
```      