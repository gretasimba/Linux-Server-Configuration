# Linux-Server-Configuration
This is Linux server installation and preparation to host my Catalog application. I secured the server from a number of attack vectors, installed and configured a Postgres, and deploeyed application onto it.The EC2 URL is 
http://ec2-13-58-96-16.us-east-2.compute.amazonaws.com/ and the local IP address is http://13.58.96.16/

# Ubuntu Linux server
Started a new Ubuntu Linux server instance on Amazon Lightsail.
Follow instructions on Udacity Project "Get started on Lightsail" page
Created "olga-first-linux" instance.
Download SSH .pem file and saved it in Vagrant folder

# Secure Server
Update and Install new versions of all currently installed packages:
 $ sudo apt-get update 
 $ sudo sudo apt-get upgrade
 
Change the SSH port from 22 to 2200
 edit /etc/ssh/sshd_config, changed port to 2200
 configure the Lightsail: added Custom TCP 2200 in Networking tab

Set Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
  $  sudo ufw status (verify inactive)
  $  sudo ufw default deny incoming
  $  sudo ufw default allow outgoing
  $  sudo ufw allow 2200/tcp
  $  sudo ufw allow www
  $  sudo ufw allow ntp
  $  sudo ufw enable
  
# SSH to Server from local Vagrant virtual machine
SSH into the "olga-first-linux" instance, specify the path to .pem (SSH) file, the port 2200, the instance user ubuntu and ip address:
  $ ssh -i PK.pem -p 2200 ubuntu@18.219.198.53
  
# Give grader access
 Create a new user account named grader.
  $ sudo adduser grader
 Give grader the permission to sudo.
 Copy 
  $ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
 And Replace ubuntu with grader using nano editor:
  $ sudo nano /etc/sudoers.d/grader
 
 
 
  
 Create an SSH key pair for grader using the ssh-keygen tool.
