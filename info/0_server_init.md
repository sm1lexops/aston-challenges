# Install Ansible

* To install Ansible, login to remote host and install ansible

> run

```sh
sudo apt-get update
sudo apt-get install -y ansible
```

> create `ansible.cfg` and other files 

```sh
ansible/
├── ansible.cfg
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── gitlab-runner.yml
├── inventory/
│   ├── hosts.ini
│   └── production/
│       └── hosts.ini
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   └── templates/
│   ├── webserver/
│   │   ├── tasks/
│   │   └── templates/
│   └── gitlab-runner/
│       ├── tasks/
│       └── templates/
├── group_vars/
    ├── all.yml
    └── webservers.yml
```

> add to `playbooks/gitlab-runner.yml`

```yml
# playbooks/gitlab-runner.yml
---
- name: Install and configure GitLab Runner
  hosts: gitlab_runners
  become: yes
  tasks:
    - name: Install required packages
      apt:
        name:
          - curl
          - apt-transport-https
          - ca-certificates
          - software-properties-common
        state: present
        update_cache: yes

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker
      apt:
        name: docker-ce
        state: present
        update_cache: yes

    - name: Add user to Docker group
      user:
        name: gitlab-runner
        groups: docker
        append: yes

    - name: Download GitLab Runner binary
      get_url:
        url: "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"
        dest: /usr/local/bin/gitlab-runner
        mode: '0755'

    - name: Create gitlab-runner user
      user:
        name: gitlab-runner
        createhome: yes
        shell: /bin/bash

    - name: Install GitLab Runner service
      command: /usr/local/bin/gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner

    - name: Start GitLab Runner service
      service:
        name: gitlab-runner
        state: started
        enabled: yes

    - name: Register GitLab Runner
      command: >
        /usr/local/bin/gitlab-runner register --non-interactive
        --url "{{ gitlab_url }}"
        --registration-token "{{ gitlab_registration_token }}"
        --executor docker
        --description "{{ gitlab_runner_name }}"
        --tag-list "docker,linux"
        --docker-image "docker:latest"
        --run-untagged
        --locked="false"
      environment:
        GITLAB_URL: "{{ gitlab_url }}"
        REGISTRATION_TOKEN: "{{ gitlab_registration_token }}"
      args:
        creates: /etc/gitlab-runner/config.toml
```
