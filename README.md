# Docker Setup for Ansible Demo

## Introduction

This is the Docker environment setup needed to execute the following repos: [Ansible-Config](https://github.com/oebinisa/Ansible-Config) and [Ansible-Modular-Config-with-Roles](https://github.com/oebinisa/Ansible-Modular-Config-with-Roles).

Basic Ansible configuration to set up a 3-tier web application architecture consisting of a load balancer tier (NGiNX), an applicaation tier - two Web servers (Apache webserver), and a database tier (MySQL server). All being accessed via a control machine/system.

- The alpine linux would be used as base image for the different containers
- The Control machine needs SSH access to the other machines

## System Structure/Architecture:

                  ─────────
                 │Control01
                 └─────────
     │────────────────│──────────────│
     │                │              │
    ──────      ──────────────      ──────
    │lb01       │App01 - App02      │db01
    └──────     └──────────────     └──────
    LoadBalancer    Webserver       Database

## Steps

1.  Create Dockerfile
    In the project root directory, create a Dockerfile with the following content:

            FROM alpine:3.18.4

2.  Login to docker:

            docker login
            Username: your_username
            Password:
            Login Succeeded

3.  Create the Docker Network:

    This custom Docker network to ensure that all containers can communicate with each other using DNS:

            docker network create mynetwork

4.  Pull the base image into the project root directory:

            pwd
            docker-setup-ansible
            docker pull alpine:3.18.4

5.  Create the following containers from the pulled image and attach them to the custom network:

        Control01:
            docker run -d --name Control01 --network mynetwork alpine:3.18.4 sleep infinity

        lb01:
            docker run -d --name lb01 --network mynetwork alpine:3.18.4 sleep infinity

        App01:
            docker run -d --name App01 --network mynetwork alpine:3.18.4 sleep infinity

        App02:
            docker run -d --name App02 --network mynetwork alpine:3.18.4 sleep infinity

        db01:
            docker run -d --name db01 --network mynetwork alpine:3.18.4 sleep infinity

6.  Install and enable SSH in the Control01 container:

        docker exec -it Control01 apk add openssh
        docker exec -it Control01 ssh-keygen -A

7.  Test SSH and DNS Resolution:

        # SSH into App01
        docker exec -it Control01 ssh App01

        # Ping App02 using its container name
        docker exec -it Control01 ping App02

        # Ping the LoadBalancer
        docker exec -it Control01 ping lb01

End.
