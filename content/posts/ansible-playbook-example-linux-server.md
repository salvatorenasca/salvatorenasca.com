---
title: "Ansible Playbook Example - Linux Server deployment with SSH Public Key"
date: 2022-12-12T11:33:23-03:00
description: "An Ansible Playbook that deploys a Linux Server"
categories:
  - Linux
  - Devops
tags:
  - Cloud
  - OnPremise
  - Ansible
  - Linux
draft: false
---

**This is an example of an Ansible Playbook to deploy a Linux Server with a Public Key for SSH.**


{{< highlight yaml >}}
---

- hosts: all
  become: true
  tasks:
  
  - name: Install and update system packages
    apt:
      name: "{{ packages }}"
      state: latest
    vars:
      packages:
        - python
        - python-apt
  
  - name: Install and configure SSH
    apt:
      name: openssh-server
    become: true
    become_user: root
    vars:
      packages:
        - openssh-server

  - name: Start and enable SSH service
    service:
      name: ssh
      state: started
      enabled: true

  - name: Add public SSH key
    authorized_key:
      user: "{{ ansible_user }}"
      key: "{{ ssh_public_key }}"
    vars:
      ssh_public_key: "ssh-rsa AAAAB...Z user@example.com"

 {{< /highlight >}}     