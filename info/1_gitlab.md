# Install Ansible

* Install Ansible on the remote host with Gitlab CI

> add to '.gitlab-ci.yml'

```yaml
stages:
  - install

variables:
  REMOTE_USER: ${REMOTE_USER}
  REMOTE_HOST: ${REMOTE_HOST}
  SSH_PRIVATE_KEY: ${SSH_PRIVATE_KEY}

install_ansible:
  stage: install
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $REMOTE_HOST >> ~/.ssh/known_hosts
  script:
    - ssh $REMOTE_USER@$REMOTE_HOST 'sudo apt update && sudo apt install -y software-properties-common'
    - ssh $REMOTE_USER@$REMOTE_HOST 'sudo apt-add-repository --yes --update ppa:ansible/ansible'
    - ssh $REMOTE_USER@$REMOTE_HOST 'sudo apt install -y ansible'
  only:
    - master
```
