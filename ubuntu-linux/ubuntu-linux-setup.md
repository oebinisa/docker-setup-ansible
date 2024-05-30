# Docker Setup for Ansible Demo

## Ubuntu Linux Setup Steps

1.  Create Dockerfile

    In the project root directory, create a Dockerfile with the following content:

            ```dockerfile
            FROM ubuntu:latest
            RUN apt-get update && apt-get install -y openssh-server sudo
            RUN mkdir /var/run/sshd
            RUN echo 'root:rootpassword' | chpasswd
            RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            EXPOSE 22
            CMD ["/usr/sbin/sshd", "-D"]
            ```

2.  Create the Docker Network:

    This custom Docker network ensures that all containers can communicate with each other using DNS:

            docker network create mynetwork

3.  Pull the base image into the project root directory:

            docker pull ubuntu:latest

4.  Create the following containers from the pulled image and attach them to the custom network:

            docker run -d --name control --network mynetwork ubuntu:latest sleep infinity
            docker run -d --name lb01 --network mynetwork ubuntu:latest sleep infinity
            docker run -d --name app01 --network mynetwork ubuntu:latest sleep infinity
            docker run -d --name app02 --network mynetwork ubuntu:latest sleep infinity
            docker run -d --name db01 --network mynetwork ubuntu:latest sleep infinity
            ```

5.  Install and enable SSH in the control container:

            ```sh
            docker exec -it control apt-get update
            docker exec -it control apt-get install -y openssh-server sudo
            docker exec -it control mkdir /var/run/sshd
            docker exec -it control sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it control sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it control service ssh start
            ```

6.  Install and enable SSH in the other containers (repeat for lb01, app01, app02, db01):

            docker exec -it app01 apt-get update
            docker exec -it app01 apt-get install -y openssh-server sudo
            docker exec -it app01 mkdir /var/run/sshd
            docker exec -it app01 sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it app01 sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it app01 service ssh start

