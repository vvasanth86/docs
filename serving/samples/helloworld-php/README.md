# Hello World - PHP sample

A simple web app written in PHP that you can use for testing.
It reads in an env variable `TARGET` and prints "Hello World: ${TARGET}". If
TARGET is not specified, it will use "NOT SPECIFIED" as the TARGET.

## Prerequisites

* A Kubernetes cluster with Knative installed. Follow the
  [installation instructions](https://github.com/knative/install/) if you need
  to create one.
* [Docker](https://www.docker.com) installed and running on your local machine,
  and a Docker Hub account configured (we'll use it for a container registry).

## Recreating the sample code

While you can clone all of the code from this directory, hello world
apps are generally more useful if you build them step-by-step. The
following instructions recreate the source files from this folder.

1. Create a new directory and cd into it:

    ````shell
    mkdir app
    cd app
    ````

1. Create a file named `index.php` and copy the code block below into it:

    ```php
    <?php
      $target = getenv('TARGET', true) ?: "NOT SPECIFIED";
      echo sprintf("Hello World: %s!\n", $target);
    ?>
    ```

1. Create a file named `Dockerfile` and copy the code block below into it.
   See [official PHP docker image](https://hub.docker.com/_/php/) for more details.

    ```docker
    FROM php:7.2.6-apache

    COPY index.php /var/www/html/

    # Run on port 8080 rather than 80
    RUN sed -i 's/80/8080/g' /etc/apache2/sites-available/000-default.conf /etc/apache2/ports.conf
    ```

1. Create a new file, `service.yaml` and copy the following service definition
   into the file. Make sure to replace `{username}` with your Docker Hub username.

    ```yaml
    apiVersion: serving.knative.dev/v1alpha1
    kind: Service
    metadata:
      name: helloworld-php
      namespace: default
    spec:
      runLatest:
        configuration:
          revisionTemplate:
            spec:
              container:
                image: docker.io/{username}/helloworld-php
                env:
                - name: TARGET
                  value: "PHP Sample v1"
    ```

## Building and deploying the sample

Once you have recreated the sample code files (or used the files in the sample folder) 
you're ready to build and deploy the sample app.

1. Use Docker to build the sample code into a container. To build and push with
   Docker Hub, run these commands replacing `{username}` with your
   Docker Hub username:

    ```shell
    # Build the container on your local machine
    docker build -t {username}/helloworld-php .

    # Push the container to docker registry
    docker push {username}/helloworld-php
    ```

1. After the build has completed and the container is pushed to docker hub, you
   can deploy the app into your cluster. Ensure that the container image value
   in `service.yaml` matches the container you built in
   the previous step. Apply the configuration using `kubectl`:

    ```shell
    kubectl apply -f service.yaml
    ```

1. Now that your service is created, Knative will perform the following steps:
   * Create a new immutable revision for this version of the app.
   * Network programming to create a route, ingress, service, and load balance for your app.
   * Automatically scale your pods up and down (including to zero active pods).

1. To find the URL and IP address for your service, use kubectl to list the ingress points in the cluster:

    ```shell
    kubectl get ing

    NAME                        HOSTS                                                                                   ADDRESS        PORTS     AGE
    helloworld-php-ingress   helloworld-php.default.demo-domain.com,*.helloworld-php.default.demo-domain.com   35.232.134.1   80        1m
    ```

1. Now you can make a request to your app to see the result. Replace
   `{IP_ADDRESS}` with the address you see returned in the previous step.

    ```shell
    curl -H "Host: helloworld-php.default.demo-domain.com" http://{IP_ADDRESS}
    Hello World: PHP Sample v1!
    ```

## Removing the sample app deployment

To remove the sample app from your cluster, delete the service record:

```shell
kubectl delete -f service.yaml
```