<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-couchbase


Example of an advanced Nosql database using the startx s2i builder [startx/sv-couchbase](https://hub.docker.com/r/startx/sv-couchbase). 
For more information on how to use this image, **[read startx couchbase image guideline](https://github.com/startxfr/docker-images/blob/master/Services/couchbase/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-couchbase
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/couchbase/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 30 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/couchbase/openshift-template-build.yml -n startx-example-couchbase
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-couchbase -f startx-couchbase-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-couchbase.git \
      -p SOURCE_BRANCH=master \
      -p COUCHBASE_USER=cbuser \
      -p COUCHBASE_PASSWORD=cbuser_pwd \
      -p COUCHBASE_BUCKETS=sxapi \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-couchbase
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc describe route -n startx-example-couchbase
  curl http://<url-route>
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-couchbase startx/sv-couchbase startx-couchbase-sample
# Run the application
docker run --rm -d -p 8777:8091 startx-couchbase-sample
# Test the sample application
telnet -h localhost 8777
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-couchbase.git couchbase-myapp
  cd couchbase-myapp
  rm -rf .git
  ```
- **Develop** and create a personalized page
  ```bash
  cat << "EOF"
  <html><head></head><body><h1>My Web Application</h1></body></html>
  EOF > index.html
  ```
- **Build** your current application with startx couchbase builder
  ```bash
  s2i build . startx/sv-couchbase:latest startx-couchbase-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d -p 8777:8091 startx-couchbase-myapp
  ```
- **Test** your application
  ```bash
  telnet -h localhost 8777
  ```
