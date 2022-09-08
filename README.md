DEVOPS PROJECT WITH  GITHUB, DOCKER, DOCKERHUB, JENKINS, ANSIBL

Objective:

Github repo will have the source code

JENKINS SERVER - code will be pushed to jenkins with the help of webhook

ANSIBLE SERVER- Jenkins server will receive the dockerfile and build the image

DOCKER HUB - the image will be pushed to docker hub

ANSIBLE SERVER - Will run the playbook and will have the ip address of the web app server. 

WEBAPP SERVER - Ansible will create the container in the webapp server

VOilA --Website will running perfectly

WEBHOOK - Will be added in github and it will whenever there is a new commit, jenkins will 
build automatically and change will reflect in the website



STEPS

1. Create 3 EC2 instances

Web server 
Ansible server
Jenkins server


2. SSH into webapp server and ansible server

3. Install docker in both instances
Ansible server have to ssh into into webapp server
without any issue

4. In the webapp server, run

vi /etc/ssh/sshd_config

scroll down and locate this 
#LoginGraceTime 2m
PermitRootLogin yes


again scroll down and change the PasswordAuthentication to yes 

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes


Then save

systemctl restart sshd

5. Repeat the process in the ansible server

6. Make ssh connection ie connect to webapp server via ansible server

* first generate public key

run this command in the ansible server


ssh-keygen

press enter and enter and enter. You will see it will generate the key


7. copy to webapp server so it can allow you do passwordless authentication

from ansible server, run


ssh-copy-id root@<private ip address of webappserver>

ssh-copy-id is a small script which copy your ssh public-key to a remote host; appending
it to your remote authorized_keys


It will ask for the password of the webapp server which you havent set

SOLUTION

Go your webapp terminal, type

passwd root
then type prefered password


Go back to ansible server and type the password

successfull display: Number of key(s) added: 1



8. ssh into the webapp server from ansible server, run

ssh root@<private ip address of webapp server>

Passwordless authentication has been done


run exit command to come of the webapp server

exit

9. make a ssh connection from jenkins server to ansible server 
secondly make ss connection from jenkins server to  webapp server

start from step 4 down to step 8 in each ssh process


JENKINS TO ANSIBLE SSH CONNECTION

Start by installing docker on jenkins and then start from step 4 -8


JENKINS TO WEBSEVER CONNECTION

start with ssh-keygen down to 8


10. Install jenkins in jenkins server

after installing successfully, login to jenkins

11. Click on new item, give name, click on freestyle project and ok


from SCM, click on git and it will demand that you provide repo URL

12. Create a new repo, push your code to repo using

git add .
git commit -m "meesage"
git remote add origin <URL OF REPO>
git branch -M main
git push -u origin main -f


and then copy the URL, go and past in jenkins configuration

No need for credential since i am using private repo

scroll down and change master to main branch

apply and save


13. Install a plugin called publish over SSH
After that go back to dashboard, manage jenkins, configure systems 

scroll down until you see publish over SSH

Enter the private key of jenkins. ie copy the private key IN jenkins terminal by running 

cat id_rsa


14. So next you have to add the 3 servers you created in AWS. 

This is to enable jenkins server know that it will connect to ansible and webapp server


Under ssh servers, click on add, type jenkins as name

Paste jenkins private ip address as hostname


username is root or whatever you used as username

click on advance and tick the box of 'Use password authentication, or use a different key'


provide the password of jenkins server

click on test configuration..it should be successful


Add Ansible server and web app server same way

Then apply and save


 15. Go to back to your project, click on configure to continue with the project

Ensure github URL is saved.

Build your job to ensure that git checkout is working

Success!

Verify the git checkout
go to jenkins server (terminal) and run

cd /var/lib/jenkins/workspace/
ls

you should see your project

cd Docker-Webapp1
ls


* Next step: Jenkins server should send the 3 files it has to ansible server

16. Go back to jenkins project, click on configure, scroll down to build step,
 select send files or execute command over ssh


SSH server should be jenkins

under Exec command, type

rsync -avh <currentworkdirofjenkins>/* root@<ip address of ansible>

the remote directory should have /opt


*Ensure chmod 777 <filename> is done for the 3 files on jenkins as well the private key for fulll permission


So this will copy all files from jenkins server, ssh into ansible and paste it there


Under SSH server, click on advance

check the box of Verbose output in console

apply and save


Build your job


17. Verify...go to ansible terminal and run

cd /opt/
you will see the files


18. Ansible is ready to then build the image and push it to dockerhub

do the following 

go to your job and add execute command over shh

use exec command to write the script that will build your image, tag it and push.

save and build the job...once successful, move to next stage


19: install ansible in ansible server


vi /etc/ansible/hosts

paste the ip address of the webserver under [web app]

Time to create ansible playbook that will build the container

cd /opt/

create or paste your ansible playbook that will run the container


19. go back to jenkins dashboard and add another execute publish over ssh

write the exec command

ansible-playbook /opt/<filename>

save and build the project

if successful, take the public ip address of your webapp server, go to the browser and you will see your website running

 






