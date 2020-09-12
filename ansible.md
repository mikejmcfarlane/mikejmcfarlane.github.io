# Ansible for the Raspberry Pi

- [Ansible for the Raspberry Pi](#ansible-for-the-raspberry-pi)
  - [Introduction](#introduction)
  - [Basics](#basics)
  - [Basic commands](#basic-commands)

## Introduction

Ansible is a simple to use and secure automation tool for managing from a few to many computers.

## Basics

* [Ansible documention](https://docs.ansible.com/ansible/latest/index.html)
* [How to Use Ansible: A Reference Guide](https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide)

## Basic commands

```bash
ansible -i hosts all -u pi -m ping
```

```bash
ansible -i hosts all -u pi -a "sudo apt update"
ansible -i hosts all -u pi -a "sudo apt -y full-upgrade"
```
