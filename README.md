# Ansible-Kubernetes-Jenkins-Helm Stack

* An Ansible role to deploy k8s cluster on any number of nodes  
* Helm charts for deploying Jenkins over k8s, and helm chart for deploying a simple Bulletin Board App.
* Jenkins pipeline for testing/building/pushing a docker image for the Bulletin Board App and deploy it on the kubernetes cluster.

# Ansible roles for deploying k8s cluster
A role to deploy a k8s cluster on a group of on-promise servers .

## Distros tested
* Ubuntu 18.04
* Debian 10.x
* CentOS / RHEL: 7.x (need some enhancements).

## Dependencies
Requires Ansible 2.6 or higher.

## ansible-vault
Use ansible-vault to encrypt sensitive info from git.

## Example Ansible Inventory file
```yaml
[master]
master-01
master-02
[node]
node-01
node-02
node-03
node-04
node-05
[kube-cluster:children]
master
node
```
## Example Playbook create-users.yml
```bash
---
- hosts: master
  gather_facts: yes
  become: yes
  vars_files:
    - group_vars/all.yml
    - group_vars/kube-cluster.yml
  roles:
    - { role: kubernetes/master, tags: master }
    - { role: cni, tags: cni }
    - { role: nfs, tags: nfs }
    - { role: helm, tags: helm }
- hosts: node
  gather_facts: yes
  become: yes
  vars_files:
    - group_vars/all.yml
    - group_vars/kube-cluster.yml
  roles:
    - { role: kubernetes/node, tags: node }
    - { role: nfs-client, tags: nfs-client }
```
## Prep
* Install Ansible
* Run Ansible Commands

## Usage
```bash
ansible-playbook site.yaml --ask-vault-pass -i hosts.ini
```
# Helm charts

## Usage

## Jenkins
```bash
helm upgrade --install [RELEASE_NAME] -n [NAMESPACE] [flags] .
```
### Or
```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install [RELEASE_NAME] jenkins/jenkins [flags]
```

## bulletinapp
```bash
helm upgrade --install [RELEASE_NAME] -n demo [flags] .
```

# Jenkins pipeline

## Dependencies
* kubernetes plugin
* kubernetes CLI plugin
* Docker Pipeline plugin
* A kubernetes cloud configured

## Usage
Just create a pipeline task and copy/paste the jenkinsfile or use git to pull it, and fire a build!
