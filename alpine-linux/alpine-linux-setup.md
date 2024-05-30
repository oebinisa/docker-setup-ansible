# Docker Setup for Ansible Demo

## Alpine Linux Setup Steps

1.  Create Dockerfile

    In the project root directory, create a Dockerfile with the following content:

            FROM alpine:latest
            RUN apk update && apk add --no-cache openssh openrc
            RUN rc-update add sshd && mkdir -p /run/openrc && touch /run/openrc/softlevel

2.  Create the Docker Network:

    This custom Docker network to ensure that all containers can communicate with each other using DNS:

            docker network create mynetwork

3.  Pull the base image into the project root directory:

            pwd
            docker-setup-ansible
            docker pull alpine:latest

4.  Create the following containers from the pulled image and attach them to the custom network:

        control:

            docker run -d --name control --network mynetwork alpine:latest sleep infinity

        lb01:

            docker run -d --name lb01 --network mynetwork alpine:latest sleep infinity

        app01:

            docker run -d --name app01 --network mynetwork alpine:latest sleep infinity

        app02:

            docker run -d --name app02 --network mynetwork alpine:latest sleep infinity

        db01:

            docker run -d --name db01 --network mynetwork alpine:latest sleep infinity

5.  Install and enable SSH in the control container:

        docker exec -it control apk add openssh
        docker exec -it control ssh-keygen -A
        docker exec -it Control01 rc-service sshd start

6.  Install and enable SSH in the other containers (repeat for lb01, app01, app02, db01):

        # app01
        docker exec -it app01 apk add openssh
        docker exec -it app01 ssh-keygen -A
        docker exec -it app01 rc-service sshd start

        # app02
        docker exec -it app01 apk add openssh
        docker exec -it app01 ssh-keygen -A
        docker exec -it app01 rc-service sshd start

        # lb01
        docker exec -it lb01 apk add openssh
        docker exec -it lb01 ssh-keygen -A
        docker exec -it lb01 rc-service sshd start

        # db01
        docker exec -it db01 apk add openssh
        docker exec -it db01 ssh-keygen -A
        docker exec -it db01 rc-service sshd start

7.  Set up SSH key-based authentication:

        # app01
        docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
        docker exec -it app01 mkdir -p /root/.ssh
        docker exec -it app01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

        # app02
        docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
        docker exec -it app02 mkdir -p /root/.ssh
        docker exec -it app02 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

        # lb01
        docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
        docker exec -it lb01 mkdir -p /root/.ssh
        docker exec -it lb01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

        # db01
        docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
        docker exec -it db01 mkdir -p /root/.ssh
        docker exec -it db01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

8.  Test SSH and DNS Resolution:

        # SSH into app01
        docker exec -it control ssh app01

        # Ping app02 using its container name
        docker exec -it control ping app02

        # Ping the LoadBalancer
        docker exec -it control ping lb01

        # Ping the Database
        docker exec -it control ping db01

End.