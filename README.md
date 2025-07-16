# badgehub-infra

This repository contains the configuration for the BadgeHub infrastructure. Currently in development.

## Overview

BadgeHub is ran as a set of Docker compose files on a Linux provisioned host. We use Ansible to bootstrap the host and then run the Docker composes. 

We have several ansible roles designed to be modular, so they may be used independently. The Ansible playbooks will be designed to run but not limtied to:
- BadgeHub API
- Keycloak for authentication

The ansible playbooks are currently designed to be ran remotely and not on the host it's supposed to be installed on, it can still be done but it has not been tested. You do need to clone the repo on the target host, if not running the ansible playbooks, to get the deployment docker compose files. 

For each compose file you have a `.env.local` for setting it up locally for development purposes. 

## Goals

The goal is to create an easy and reproducible recipe to procure BadgeHub infrastructure that will reliably host the services needed for WHY2025 and any future events held by any other parts of the community.

## Requirements 

### Software

- Ansible core 2.17.1 minimum
- Debian target host with internet access

## Setting up the Linux host to run Badgehub

Check [ansible.md](./docs/ansible.md) on how to provision your host if you do not already do so. 

## Running it locally with only docker compose

First create the docker network

`docker network create badgehub_network`

### Networking

BadgeHub application ,Keycloak and Postgres databases all see eachother within the docker network. The Docker compose files then expose the services to be latched on by a reverse proxy of some sorts or be used independently. 

### Keycloak
If you wish to have working authentication, you must use Keycloak, check `docs/` for Keycloak installation and configuration details. 

### Badgehub 

Run badgehub app with postgres database.

#### Local Development

`cd badgehub && docker compose --env-file .env.local up -d`

#### Production

`cd badgehub && docker compose up -d`

### Recommended Hardware Specifications

There is currently no real world test case on how many resources does the server need as it is in early development. A good guesstimate is:

#### For Test VM:
- 2 vCPU minimum
- 2-4GB RAM minimum 
- 20-30GB disk space

#### For Actual Server:
- 4 vCPU minimum
- 4GB RAM minimum
- 50GB disk space

## Contribution Guide

At the moment, one recommendation is to use `ansible-lint` to lint the playbooks. You may create a fork with a different branch name and then create a PR explaining the changes. For any other questions or issues, you may open an issue or discussion.

## License 

MIT
