# Docker Setup for Ansible Demo

## Alpine Linux Setup Steps

1.  Create Dockerfile

    In the project root directory, create a Dockerfile with the following content:

            FROM alpine:latest

            # Install openssh and openrc
            RUN apk update && apk add --no-cache openssh openrc

            # Set up OpenRC
            RUN rc-update add sshd && mkdir -p /run/openrc && touch /run/openrc/softlevel
           
             # Generate SSH host keys
            RUN ssh-keygen -A             
            
            # Expose SSH port
            EXPOSE 22                      
            
            # Start SSH daemon
            CMD ["/usr/sbin/sshd", "-D"]   

2.  Build the Docker image from the Dockerfile.

            docker build -t alpine-ssh .

3.  Create the Docker Network:

    This custom Docker network to ensure that all containers can communicate with each other using DNS:

            docker network create mynetwork

4.  Create the following containers from the pulled image and attach them to the custom network:

            docker run -d --name control --network mynetwork alpine-ssh sleep infinity
            docker run -d --name lb01 --network mynetwork alpine-ssh sleep infinity
            docker run -d --name app01 --network mynetwork alpine-ssh sleep infinity
            docker run -d --name app02 --network mynetwork alpine-ssh sleep infinity
            docker run -d --name db01 --network mynetwork alpine-ssh sleep infinity

5.  Setup and generate an SSH key on the control container: (AWS EC2 - Start here*)

        docker exec -it control apk add openssh
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
        docker exec -it app01 apk add openssh
        docker exec -it app01 ssh-keygen -A
        docker exec -it app01 rc-update add sshd
        docker exec -it app01 openrc default

        # app02
        docker exec -it app02 apk add openssh
        docker exec -it app02 ssh-keygen -A
        docker exec -it app02 rc-update add sshd
        docker exec -it app02 openrc default

        # lb01
        docker exec -it lb01 apk add openssh
        docker exec -it lb01 ssh-keygen -A
        docker exec -it lb01 rc-update add sshd
        docker exec -it lb01 openrc default

        # db01
        docker exec -it db01 apk add openssh
        docker exec -it db01 ssh-keygen -A
        docker exec -it db01 rc-update add sshd
        docker exec -it db01 openrc default

8.  Test SSH and DNS Resolution:

    Ensure that SSH and DNS resolution are working correctly.

        # SSH into app01
        docker exec -it control ssh app01

        # Ping app02 using its container name
        docker exec -it control ping app02

        # Ping the LoadBalancer
        docker exec -it control ping lb01

        # Ping the Database
        docker exec -it control ping db01

End.