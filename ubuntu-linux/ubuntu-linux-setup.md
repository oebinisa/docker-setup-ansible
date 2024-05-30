# Docker Setup for Ansible Demo

## Ubuntu Linux Setup Steps

1.  Create Dockerfile

    In the project root directory, create a Dockerfile with the following content:

            FROM ubuntu:latest
            RUN apt-get update && apt-get install -y openssh-server sudo
            RUN mkdir /var/run/sshd
            RUN echo 'root:rootpassword' | chpasswd
            RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            EXPOSE 22
            CMD ["/usr/sbin/sshd", "-D"]

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

5.  Install and enable SSH in the control container:

            docker exec -it control apt-get update
            docker exec -it control apt-get install -y openssh-server sudo
            docker exec -it control mkdir /var/run/sshd
            docker exec -it control sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it control sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it control service ssh start

6.  Install and enable SSH in the other containers (repeat for lb01, app01, app02, db01):

            # app01
            docker exec -it app01 apt-get update
            docker exec -it app01 apt-get install -y openssh-server sudo
            docker exec -it app01 mkdir /var/run/sshd
            docker exec -it app01 sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it app01 sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it app01 service ssh start

            # app02
            docker exec -it app02 apt-get update
            docker exec -it app02 apt-get install -y openssh-server sudo
            docker exec -it app02 mkdir /var/run/sshd
            docker exec -it app02 sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it app02 sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it app02 service ssh start

            # lb01
            docker exec -it lb01 apt-get update
            docker exec -it lb01 apt-get install -y openssh-server sudo
            docker exec -it lb01 mkdir /var/run/sshd
            docker exec -it lb01 sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it lb01 sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it lb01 service ssh start

            # db01
            docker exec -it db01 apt-get update
            docker exec -it db01 apt-get install -y openssh-server sudo
            docker exec -it db01 mkdir /var/run/sshd
            docker exec -it db01 sh -c "echo 'root:rootpassword' | chpasswd"
            docker exec -it db01 sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
            docker exec -it db01 service ssh start

7.  Set up SSH key-based authentication:

            # app01
            docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
            docker exec -it app01 mkdir -p /root/.ssh
            docker exec -it app01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

            # app02
            docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
            docker exec -it app02 mkdir -p /root/.ssh
            docker exec -it app02 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

            # ld01
            docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
            docker exec -it ld01 mkdir -p /root/.ssh
            docker exec -it ld01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

            # db01
            docker exec -it control ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
            docker exec -it db01 mkdir -p /root/.ssh
            docker exec -it db01 sh -c "echo '$(docker exec control cat /root/.ssh/id_rsa.pub)' >> /root/.ssh/authorized_keys"

8.  Test SSH and DNS Resolution:

            # SSH into app01
            docker exec -it control ssh root@app01

            # Ping app02 using its container name
            docker exec -it control ping app02

            # Ping the LoadBalancer
            docker exec -it control ping lb01            

End.

