# Docker Setup for Ansible Demo

## Ubuntu Linux Setup Steps

1.  Create Dockerfile

    In the project root directory, create a Dockerfile with the following content:

            FROM ubuntu:latest

            # Install OpenSSH server
            # RUN apt-get update && apt-get install -y openssh-server
            RUN apt-get install -y openssh-server

            # Create the SSH directory and generate SSH host keys
            RUN mkdir /var/run/sshd && ssh-keygen -A

            # Expose SSH port
            EXPOSE 22

            # Start SSH daemon
            CMD ["/usr/sbin/sshd", "-D"]


2.  Build the Docker image from the Dockerfile.

            docker build -t ubuntu-ssh .


3.  Create the Docker Network:

    This custom Docker network ensures that all containers can communicate with each other using DNS:

            docker network create mynetwork


4.  Create the following containers from the created image and attach them to the custom network:

            docker run -d --name control --network mynetwork ubuntu-ssh sleep infinity
            docker run -d --name lb01 --network mynetwork ubuntu-ssh sleep infinity
            docker run -d --name app01 --network mynetwork ubuntu-ssh sleep infinity
            docker run -d --name app02 --network mynetwork ubuntu-ssh sleep infinity
            docker run -d --name db01 --network mynetwork ubuntu-ssh sleep infinity


5.  Setup and generate an SSH key on the control container:

            docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa


6.  Set up SSH key-based authentication:

    Distribute the public key on control to other containers.

            # app01
            docker exec -it app01 mkdir -p /root/.ssh
            docker exec -it app01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"
            docker exec -it control sh -c 'mkdir -p /root/.ssh && touch /root/.ssh/known_hosts && ssh-keyscan -H app01 >> /root/.ssh/known_hosts'

            # app02
            docker exec -it app02 mkdir -p /root/.ssh
            docker exec -it app02 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"
            docker exec -it control sh -c 'mkdir -p /root/.ssh && touch /root/.ssh/known_hosts && ssh-keyscan -H app02 >> /root/.ssh/known_hosts'

            # lb01
            docker exec -it lb01 mkdir -p /root/.ssh
            docker exec -it lb01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"
            docker exec -it control sh -c 'mkdir -p /root/.ssh && touch /root/.ssh/known_hosts && ssh-keyscan -H lb01 >> /root/.ssh/known_hosts'

            # db01
            docker exec -it db01 mkdir -p /root/.ssh
            docker exec -it db01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"
            docker exec -it control sh -c 'mkdir -p /root/.ssh && touch /root/.ssh/known_hosts && ssh-keyscan -H db01 >> /root/.ssh/known_hosts'


7.  Start SSH service on the other containers:

            # app01
            docker exec -it app01 service ssh start

            # app02
            docker exec -it app02 service ssh start

            # lb01
            docker exec -it lb01 service ssh start

            # db01
            docker exec -it db01 service ssh start


8.  Test SSH and DNS Resolution:

    Ensure that SSH and DNS resolution are working correctly.

            # SSH into app01
            docker exec -it control ssh app01

            # SSH into app01
            docker exec -it control ssh app02

            # Ping the LoadBalancer
            docker exec -it control ping lb01

            # Ping the Database
            docker exec -it control ping db01
            

End.

