# Aston DevOps challenges

*by Aleksey Smirnov*

1. Using Gitlab to all challenges

2. Configure gitlab-runner on server1

3. Deploy to the server2 wordpress stage prod with docker-compose with automation build and deploy

4. Install Zabbix to the server1 and configure alerting problem from wordpress and server1 server2 with notification to the telegram group

5. Commit to gitlab.com

* How to decrease time to build when deploying services, and optimise gitlab-runner for works with docker

  - Use dind (docker in docker)

  - Use Docker Cache

  - Optimize Dockerfile (multi-stage build, minimize layers, order to run instructions)

  - Resource management

* Launching ansible directly via runner

* Run all in multistage CI/CD


## Configure Remote Server

* Install Ansible on a remote server and configure with playbook docker, docker-compose, gitlab-runner

  - Add sudo usermod -aG docker gitlab-runner

  - - name: Add current user to Docker group
  command: "usermod -aG docker {{ ansible_user_id }}"
  become: yes

  - - name: Add gitlab-runner user to Docker group
  command: "usermod -aG docker gitlab-runner"
  become: yes

  - - name: Register GitLab Runner
      become: yes
      become_user: root
      environment:
        GITLAB_URL: "{{ gitlab_url }}"
        REGISTRATION_TOKEN: "{{ gitlab_registration_token }}"
      command: >
        /usr/local/bin/gitlab-runner register --non-interactive
        --url "{{ gitlab_url }}"
        --registration-token "{{ gitlab_registration_token }}"
        --executor docker
        --description "{{ gitlab_runner_name }}"
        --tag-list "aston-runner"
        --docker-image "docker:latest"
        --run-untagged
        --locked="false"
      args:
        creates: /etc/gitlab-runner/config.toml



* Install docker and docker-compose on a remote server

[First step instruction](info/0_install_docker.md)

## 1. Deploy and Install Gitlab Runner

[Install and configure Gitlab runners](https://docs.gitlab.com/ee/tutorials/create_register_first_runner/)


[Second step instruction](info/1_install_ansible.md)

## 1. Install Ansible with Gitlab CI
## 2. Install Docker with Ansible


## 3. 

* Deploy and register Gitlab runner on a remote server VM using Ansible 

[Third step instruction](info/2_install_gitlab_runner.md)

* Create an Ansilbe playbook for deploying and register ansible on a remote host
