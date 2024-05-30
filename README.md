# Docker Setup for Ansible Demo

## Introduction

This is the Docker environment setup needed to execute the following repos: [Ansible-Config](https://github.com/oebinisa/Ansible-Config) and [Ansible-Modular-Config-with-Roles](https://github.com/oebinisa/Ansible-Modular-Config-with-Roles).

Basic Ansible configuration to set up a 3-tier web application architecture consisting of a load balancer tier (NGiNX), an applicaation tier - two Web servers (Apache webserver), and a database tier (MySQL server). All being accessed via a control machine/system.

- The alpine and Ubuntu linux would be used as base image for the different containers
- The two setups will be outlined below
- The Control machine needs SSH access to the other machines

## System Structure/Architecture:

                  ─────────
                 │control
                 └─────────
     │────────────────│──────────────│
     │                │              │
    ──────      ──────────────      ──────
    │lb01       │app01 - app02      │db01
    └──────     └──────────────     └──────
    LoadBalancer    Webserver       Database



## Alpine Linux Setup Steps

 - See alpine-linux folder


## Ubuntu Linux Setup Steps

 - See Ubuntu-linux folder

End.
