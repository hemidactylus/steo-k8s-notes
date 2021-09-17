# First steps

Welcome to "Your first nontrivial architecture on Kubernetes"!

This is a reproducible account of my first baby steps to take some simple
"application" and deploy it within K8s. The focus is on the architecture,
so don't be disappointed if the applications themselves are less than your
typical "hello world".

### Pre-requisites

You need a [dockerhub](https://hub.docker.com/) account,
Docker installed and a Kubernetes cluster accessible with `kubectl`.

In particular, Docker should be aware of your dockerhub
account. In practice make sure the `docker login` command
returns something like `Login Succeeded`:
it will then be able to push container images there for you.

## Step 1: dockerize something

First step is, you have an application running as stand-alone
and you want to pack it into a Docker image.

### Develop your app

The application is a very simple Python app written in Flask.
We will use Python 3.6.
The project is made by two files, a `requirements.txt` with just:

    Flask==2.0.1

and an `app.py` with:

```
from flask import Flask

app = Flask('DApp')

@app.route('/')
def home():
    return {'a': 1}

@app.route('/info')
def info():
    return {'info': True}

app.run(host='0.0.0.0')
```

Create these two files in a new directory. To have this running you may
want to keep a dedicated Python virtualenv. Anyway, once you install your
dependencies with

    pip install -r requirements.txt

you can launch the API with

    python app.py

At this point you can test it works with something like

    curl localhost:5000/
    curl localhost:5000/info

_Note: not fit for production! You would need to use something like
uwsgi to handle your API's execution. This is, however, out of scope here._

Close the app with Ctrl-C.

### Dockerize it

We now pack the whole app (its requirement, runtimes, dependencies, code and
everything) into a neat Docker image.

To do that we create a `Dockerfile` file in the same directory with contents:

```
FROM python:3.6

WORKDIR /code

COPY requirements.txt /requirements.txt

RUN pip install -r /requirements.txt

COPY . /code

EXPOSE 5000

CMD ["python3", "app.py"]
```

This says: we start with a base image having Python 3.6, we copy the req file
and use it to install dependencies, we copy the whole dir (with the app) to
the image; the container will expose port 5000, and when it starts it has to
run the command we ran manually earlier.

Now let's build the image and give it the name `f-webapp-1` and a tag `0.1`:

    docker build . -t f-webapp-1:0.1

This can take several minutes and at the end you will see the image with

    docker image ls

Now we push the image to dockerhub. First let us tag it with a prefix
that is our dockerhub username (note the name can change: we are effectively
giving multiple labels to the same image):

    docker tag f-webapp-1:0.1 stlottini/first-app:1.0

Finally let us push the image to dockerhub:

    docker push stlottini/first-app:1.0

You can now go to dockerhub and will see it there.

### Pulling and running

Note: You could run the following from another machine as well.
If you are using the same machine you pushed from, the pull is not
actually performed as everything is locally cached so the command

    docker pull stlottini/first-app:1.0

will be instantaneous.

Anyway, now you have the image and can start a container with it:

    docker run -p 5000:5000 stlottini/first-app:1.0

Since port 5000 is mapped to the outside, once this runs you can easily test
the container is runnig properly with the very same `curl` commands as before.

## Step 2: deploy to Kubernetes

Now it's time to take the image created above and wrap it as a pod in K8s,
further wrapped into a deployment.

First ensure you have a K8s cluster running and that kubectl can reach it.

Next we create a manifest describing the whole system, `deployment.yaml`, with
contents:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: f-webapp
  name: f-webapp
spec:
  selector:
    matchLabels:
      run: f-webapp
  replicas: 1
  minReadySeconds: 120
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: f-webapp
    spec:
      containers:
      - image: stlottini/first-app:1.0
        name: f-webapp
---
apiVersion: v1
kind: Service
metadata:
  name: s-f-webapp
  labels:
    app: web
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 31000
  selector:
    run: f-webapp
```

There are two resources here: a deployment, based on the container image
we created earlier and a NodePort service, that is attached to the
deploy and makes it possible to reach it. Trying to get to individual pods
"from outside" (or from other parts of the cluster, for that matter)
is bad practice since it is not resilient to pods being replaced or
deployments scaling out or in. Indeed, the Service resource in K8s
serves exactly the purpose of providing a stable abstraction to reach pods.

Note that in the NodePort we have mapped the container port 5000 to port 31000.

Note that the service and the deployment are bound to each other through the
selector label `f-webapp`.

Now deploy:

    kubectl apply -f deployment.yaml

You can check with `kubectl get pod` or `kubectl get deploy`, maybe
appending `--watch` to these commands.

After about one minute, the pod will be `Running` and the deploy will
have `READY: 1/1`.

Also `kubectl get svc` will give the service we created.

Now get the IP of the cluster node with

    kubectl get nodes -o wide

which returns e.g. `172.20.0.2` as `INTERNAL-IP`, and try to reach the app
running in the pod from outside:

    curl 172.20.0.2:31000/info

If everything works, that's it. Otherwise, good luck with

    kubectl logs <pod name from kubectl get pod>

## Step 3: a more ambitious deploy

Time to create something more complex made by several parts.

TODO