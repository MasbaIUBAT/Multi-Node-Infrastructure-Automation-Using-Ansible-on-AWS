Structure:



01-infrastructure-and-ansible-setup.md



1\. Project Overview

2\. Project Goal

3\. Architecture Diagram

4\. AWS EC2 Infrastructure

5\. Operating System Details

6\. Network Design

7\. User Configuration

8\. SSH Passwordless Authentication

9\. Sudo Configuration

10\. Ansible Installation

11\. Ansible Configuration File

12\. Inventory Configuration

13\. Ansible Connectivity Test

14\. Problems Faced and Solutions

15\. Current Status

16\. Next Phase



The document should explain:



1\. Project Overview



Example:



\# CloudForge: Multi-Node Infrastructure Automation Using Ansible on AWS



\## Phase 1: Infrastructure and Ansible Setup



This phase focuses on building the AWS infrastructure, configuring secure SSH communication, and preparing Ansible automation between the control node and managed nodes.

2\. Architecture Diagram



Use:



&#x20;                        AWS VPC



&#x20;                   Master Node

&#x20;                   Ubuntu Server

&#x20;                   Ansible Controller



&#x20;                   Public IP:

&#x20;                   98.93.67.178



&#x20;                   Private IP:

&#x20;                   172.31.28.51



&#x20;                         |

&#x20;                         |

&#x20;             SSH Key Authentication

&#x20;                         |

&#x20;       --------------------------------

&#x20;       |                              |



&#x20;     Node 1                         Node 2



&#x20;Amazon Linux 2023              Amazon Linux 2023



&#x20;Private IP:                    Private IP:

&#x20;172.31.19.5                    172.31.30.81



&#x20;User: ubuntu                   User: ubuntu



&#x20;Managed Node                   Managed Node

3\. AWS Infrastructure



Document:



Instance	Role	OS	Private IP

Master	Ansible Control Node	Ubuntu	172.31.28.51

Node 1	Managed Node	Amazon Linux 2023	172.31.19.5

Node 2	Managed Node	Amazon Linux 2023	172.31.30.81

4\. Why SSH Passwordless Authentication?



Explain:



Ansible communicates through SSH.

Manual password login cannot support automation.

Public/private key authentication allows secure communication.



Flow:



Master Node



Private Key

&#x20;    |

&#x20;    |

SSH Authentication

&#x20;    |

&#x20;    |

Public Key



Node 1 / Node 2

authorized\_keys

5\. Commands Used

Generate SSH key



Master:



ssh-keygen



Check public key:



cat \~/.ssh/id\_rsa.pub

Create user on nodes



Node:



useradd -m ubuntu



Add sudo:



usermod -aG wheel ubuntu



Verify:



groups ubuntu

Configure SSH

mkdir -p /home/ubuntu/.ssh



nano /home/ubuntu/.ssh/authorized\_keys



Permission:



chown -R ubuntu:ubuntu /home/ubuntu/.ssh



chmod 700 /home/ubuntu/.ssh



chmod 600 /home/ubuntu/.ssh/authorized\_keys

6\. Ansible Installation



Master:



ansible --version



Output:



ansible \[core 2.20.1]

7\. Inventory Configuration



File:



inventory



Content:



\[nodes]



node1 ansible\_host=172.31.19.5 ansible\_user=ubuntu



node2 ansible\_host=172.31.30.81 ansible\_user=ubuntu

8\. Ansible Configuration



File:



ansible.cfg



Content:



\[defaults]



inventory = inventory

remote\_user = ubuntu

host\_key\_checking = False

9\. Connectivity Test



Command:



ansible nodes -m ping



Result:



node1 | SUCCESS => pong



node2 | SUCCESS => pong

10\. Problems Faced



Document your real problems:



Problem 1

ansible: command not found



Cause:

Command was executed on Node instead of Master.



Solution:

Run Ansible commands only from Master Node.



Problem 2

Permission denied (publickey)



Cause:

SSH key was not correctly configured.



Solution:

Check:



/home/ubuntu/.ssh/authorized\_keys

Problem 3

cat: /root/.ssh/id\_rsa.pub: No such file



Cause:

Checked key from Node instead of Master.



Solution:

SSH key exists only on the machine that created it.



Current Status



Completed:



✅ AWS EC2 infrastructure

✅ SSH passwordless communication

✅ Ansible installation

✅ Inventory setup

✅ Ansible ping test



Next Phase:



➡️ Ansible Ad-hoc Commands

