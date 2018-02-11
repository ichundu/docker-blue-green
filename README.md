Exercise 2: Docker
==================

Create a Docker-Compose multi-container application, that acheives the following goals:

- Hosts a simple html page on two separate nginx containers.
- The html page should contain the word *"blue"* on one container and the word *"green"* on the other.
- Place a HAProxy loadbalancer container in front of the nginx containers.
- Enable a method to view HAProxy statistics via web interface.


Solution:
=========

1. Spinup a Ubuntu 16.04 or RHEL 7 (or CentOS 7) box.
2. Install `docker`, `docker-compose` and `git`:

    - Ubuntu 16.04:

        ```bash
        sudo apt update && sudo apt install -y docker.io docker-compose git
        ```

    - RHEL 7, docker-compose is not available in the official repositores so we must install the EPEL repo, refer to ["How to install docker-compose on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-centos-7):

        ```bash
        sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
        sudo yum install -y python-pip docker-compose git
        sudo yum upgrade -y python*
        ```

        Disable SELinux:

        ```bash
        sudo setenforce 0
        ```

3. Start the docker daemon

    ```
    sudo systemctl start docker
    ```

4. Clone this repository:

    ```bash
    git clone https://github.com/ichundu/docker-blue-green.git
    ```

    Navigate to the repo direcotry:

    ```bash
    cd docekr-blue-green
    ```

5. With only one command we will download and run two `nginx` and one `haproxy` containers as required:

    ```bash
    sudo docker-compose up
    ```

6. Finally access the service at:

    http://*host_ip_address*

    HAProxy statistics:

    http://*host_ip_address*/stats

    
How this works:
===============

There are three direcotries in this repo: `blue`, `green` and `haproxy`.
Each directory contains configuration files for the containers that we'll be running. They are attached as volumes to the running containers.

- The `blue` and `gree` webapps are docker container based on the [`nginx:stable-alpine`](https://hub.docker.com/_/nginx/) base images.
- The `haproxy` container is the loadbalancer that proxies http requests to the nginx containers in a roundrobin fashion. The backends configuration in `haproxy.cfg` looks like this:

    ```
    backend blue-green
        mode http
        balance roundrobin
        server blue blue:80 check
        server green green:80 check
    ```

    The good thing about `docker-compose` is that it's containers can resolve the service names so we don't need to hardcode IP addresses to the configuration files, so we are using `blue:80` and `green:80` instead of IP addresses to reach the backend containers.

    We have also configured HAProxy to show the loadbalancer statistics at the `/stats` uri:

    ```conf
    stats enable 
    stats uri /stats   
    stats refresh 5s 
    ```
