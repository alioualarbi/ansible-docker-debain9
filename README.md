# Ansible Playbook Docker Image on Debian 9
Docker Image of Ansible for executing ansible-playbook command against an externally mounted set of Ansible playbooks.
## Test
```
$ docker run dockeralioua/ansible-debain9  ansible-playbook --version
ansible-playbook 2.9.1
  config file = None
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python2.7/dist-packages/ansible
  executable location = /usr/local/bin/ansible-playbook
  python version = 2.7.13 (default, Sep 26 2018, 18:42:22) [GCC 6.3.0 20170516]
```
## Running Ansible Playbook
```
$ docker run --rm -it -v PATH_TO_LOCAL_PLAYBOOKS_DIR:/ansible/playbooks dockeralioua/ansible-debain9:latest PLAYBOOK_FILE
```
## Ansible Helper wrapper

Shell script named ansible_helper that wraps a Docker image containing Ansible:
```
$ docker run --rm -it \
  -v ~/.ssh/id_rsa:/root/.ssh/id_rsa \
  -v ~/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub \
  -v $(pwd):/ansible/playbooks \
  -v /var/log/ansible/ansible.log \
  dockeralioua/ansible-debain9 "$@"
  ```
Point the above script to any inventory file so that we can execute any Ansible command on any host, e.g.
```
./ansible_helper play playbooks/create_user.yml -i inventory/staging -e 'some_var=some_value'
```
## SSH Keys
If Ansible is interacting with external machines, you'll need to mount an SSH key pair for the duration of the play:
```
$ docker run --rm -it \
    -v ~/.ssh/id_rsa:/root/.ssh/id_rsa \
    -v ~/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub \
    -v $(pwd):/ansible/playbooks \
    dockeralioua/ansible-debain9 machine.yml
```
## Ansible Vault

If you've encrypted any data using Ansible Vault, you can decrypt during a play by either passing --ask-vault-pass after the playbook name, or pointing to a password file. For the latter, you can mount an external file:
```
docker run --rm -it -v $(pwd):/ansible/playbooks \
    -v ~/.vault_pass.txt:/root/.vault_pass.txt \
    dockeralioua/ansible-debain9 \
    site.yml --vault-password-file /root/.vault_pass.txt
```
Note: the Ansible Vault executable is embedded in this image. To use it, specify a different entrypoint:
```
docker run --rm -it -v $(pwd):/ansible/playbooks --entrypoint ansible-vault \
  dockeralioua/ansible-debain9 encrypt FILENAME
  ```
