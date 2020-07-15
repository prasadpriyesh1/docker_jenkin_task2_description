# Docker jenkins task2 description
## Task 2-
1.  Create container image that’s has Jenkins installed using dockerfile
2.  When we launch this image, it should automatically starts Jenkins service in the container.
3.  Create a job chain of  **job1, job2, job3**  and  **job4** using build pipeline plugin in Jenkins
4.  **Job1**  : Pull the Github repo automatically when some developers push repo to Github.
5.  **Job2**  : By looking at the code or program file, Jenkins should automatically start the respective language interpreter install image container to deploy code ( eg. If code is of PHP, then Jenkins should start the container that has PHP already installed ).
6.  **Job3** : Test your app if it is working or not.
7.  **Job4**  : if app is not working , then send email to developer with error messages.
8.  Create One extra job  **job5**  for monitor : If container where app is running. fails due to any reson then this job should automatically start the container again.

## Problem 1

I created a dockerfile to install jenkins and docker inside the container. Since we cannot use systemctl inside docker container I cam up with an alternative to start jenkins and docker.
** note docker will be started during container creation
FROM centos


RUN yum install git -y
RUN yum install sudo -y 

RUN curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | tee /etc/yum.repos.d/jenkins.repo

RUN rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key


RUN yum install java-11-openjdk.x86_64 -y
RUN yum install jenkins -y

RUN yum install yum-utils device-mapper-persistent-data lvm2 -y

RUN yum-config-manager --add-repo  https://download.docker.com/linux/centos/docker-ce.repo

RUN yum -y install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm


RUN yum install -y docker-ce

CMD /etc/alternatives//java -Dcom.sun.akuma.Daemon=deamonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 handlerCountMax=100 --handlerCountMaxIdle=20 


To build this docker file we used 
> docker build --tag jen_doc_img .

This will automatically start jenkins inside the container


Build Command-
> docker build --tag jen_doc_img:v1 .

## Run the above created image
Main problems to achieve the above task

 1. Expose jenkins port to out world
 2. Expose some more ports that may be required later
 3. run docker inside container(actually I linked my docker in the container to docker in my base OS )
    * use -v to link both docker files
 4. setting password for jenkins by manually merging the initialAdminPassword file to that of the base os to access the password
    * Use -v to link both jenkins password folder
 

Final Run command as follows- 
> docker run --network bridge -p 8082:8080 -p 8083:8083 -p 8084:8084 -p 8085:8085 -p 8086:8086 -v /var/lib/jenkins/secrets/initialAdminPassword:/var/lib/jenkins/secrets/initialAdminPassword -v /var/run/docker.sock:/var/run/docker.sock -v /root/docker_jenkin_code/:/var/www/html/ --name jenkin_os jen_doc_img:v1

 

## Setup Jenkins 
Now when you got to brower and access the 8082 port number 
this will run jenkins. which will ask the initial password.

![jenkins](https://github.com/narayanhari/JenkinsDocker_Task2/blob/master/jen.png)

To find this open Host OS and type
> cat /var/lib/jenkins/secrets/initialAdminPassword

Copy this password and paste it in the password field.

Now change the password and setup is done

## Problem 2

### Jenkin jobs
1. get_code_doc_jen
2. check_code
3. test_site

## get_code_doc_jen
This job fetches the data from github and pastes it to the /var/www/html/ folder of our container
>note: /var/www/html/ folder is mounted to /root/docker_jenkin_code/ folder in base os.


![job1_img1](https://github.com/prasadpriyesh1/docker_jenkin_task2_description/blob/master/Screenshot%20(96).png)
![job1_img2](https://github.com/prasadpriyesh1/docker_jenkin_task2_description/blob/master/Screenshot%20(97).png)

## check_code
This code uses linux scripting commands to check the type of code files and launches the required container image.
This will automatically execute after get_code_doc_jen job.


![job2_img1](https://github.com/prasadpriyesh1/docker_jenkin_task2_description/blob/master/Screenshot%20(91).png)
![job2_img2](https://github.com/prasadpriyesh1/docker_jenkin_task2_description/blob/master/Screenshot%20(92).png)
