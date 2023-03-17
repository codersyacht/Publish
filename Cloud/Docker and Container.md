## Docker and Container

**Preparing the system**

1. Uninstall docker if it already exists in the system.<br/>
`sudo yum remove docker-ce docker-ce-cli containerd.io`<br/>
This is not a mandatory step. User can run the above command in case it is required to remove the existing docker installation.<br/>
2. Install yum utils.<br/>
`sudo yum install -y yum-utils`<br/>

**Installation of Docker**

Steps:<br/>
1. Login to the operating system and execute the following command to add the docker repository:<br/>
`sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`<br/>

2. Install docker
`sudo yum install docker-ce docker-ce-cli containerd.io`<br/>

The installation will prompt for user input at a couple of places before the installation completes. Once the above command is executed, check if the installation process completes successfully.<br/>

3. Execute the following commands to start docker service:<br/>
`sudo systemctl start docker`<br/>

4. Test the installation by executing the following commands:<br/>
To check for the list of images available in the local repository:<br/>
`docker images`<br/>
To check if there are any active container:<br/>
`docker ps`<br/>
To check if there are any exited containers:<br/>
`docker ps -a`<br/>
For now both the above commands would return no values.<br/>

<img width="1000" alt="image" src="https://user-images.githubusercontent.com/93929892/184942280-a9f7f562-8a0b-48ed-bd4a-d4af1f663154.png">

**Creating a CentOS based Docker container**

1. Create a directory called LabSession1 </br>

2. If vi editor is not available, install using the following command:</br>
`yum -y install vim`</br>

3. create a file called Dockerfile using vi command under LabSession1 directory. Command to launch vi editor:</br>
`vi Dockerfile`</br>
Type i to enter into edit mode.</br>
Enter the following:</br>
`FROM centos` </br>
Save the file by entering Esc and then type  :wq </br>
Now the file Dockerfile contain one line - `FROM centos`. </br>

4. Execute the following command to build an image.</br>
`docker build .` </br>
If successful, you will see the following output in the terminal.</br>
`Successfully built a2a15febcdf3` </br>

5. Execute the following command to check the image availability in the local repository. </br>
`docker images` </br>

6. Execute the following command to run the images into a container: </br>
`docker run -idt <image-id>` </br>
If the container is up and running, you will see a similar status showing “Up” for the image.

<img width="524" alt="image" src="https://user-images.githubusercontent.com/93929892/185033791-09075bdf-2c6d-4aac-aff8-2c7229cc2255.png">


**Installing Java in the container**

1. Login into the docker container using the following command:</br>
`docker exec -it <container-id> bash`</br>
If successful, you will be directed into the container console. </br>

2. Type pwd to check the present working directory. Output should be root – “/”.</br>

3. Install jdk on the local container.</br>
First command to execute:</br>
`yum install -y yum-utils`</br>
Followed by:</br>
`yum install java-1.8.0-openjdk`
run `java -version`. If installed successfully, you will see:
<img width="694" alt="image" src="https://user-images.githubusercontent.com/93929892/185534463-7305def8-a941-4b51-8197-8d8b0d29ade5.png">

4. Install unzip tool using the following command:<br/>
`yum install zip`

5. Exit from the container by entering exit command.<br/>

6. Save the docker container by executing the following command:<br/>
`docker commit <container-id>`

**Installing Liberty server in the container**

1. Download latest WebSphere liberty server from the following location:<br/>
https://developer.ibm.com/wasdev/downloads/#asset/runtimes-wlp-webProfile8 <br/>

2. Copy the liberty zip, wlp-webProfile8-19.0.0.9.zip into the base centos system.<br/>

3. Alternately, the liberty zip can be downloaded directly into the base system using the following curl command: <br/>
`curl https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/wasdev/downloads/wlp/20.0.0.7/wlp-webProfile8-20.0.0.7.zip --output wlp-webProfile8-20.0.0.7.zip`

4. Copy the zip from the base OS to the container OS. Execute the following command:<br/>
`docker cp ./wlp-webProfile8-19.0.0.9.zip 336c881a0ec6:/wlp.zip`

5. Enter into the container by once again executing:<br/>
`docker exec -it <container-id> bash` <br/>
Execute `ls` command. You will see wlp.tar file present.

6. unzip the tar using the following command:<br/>
`unzip wlp.zip`<br/>
You will see wlp folder present.

7. Navigate to wlp/bin directory using cd command. `cd wlp/bin`

8. Execute the following command to create a server instance in liberty.<br/>
`./server create AppServer`

9. navigate to /wlp/usr/servers/AppServer.

10. Edit the server.xml using the vi editor.<br/>
Add host attribute to httpEndpoint element. Example:<br/>
`<httpEndpoint id="defaultHttpEndpoint" host=”*”`<br/>
Save and exit vi editor.

11. Exit the container by typing `exit` and save the container by committing.<br/>
`docker commit <container-id>`

12. Download sample application from the following location:<br/>
https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/<br/>
`curl https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/sample.war --output sample.war`

13. Copy the sample.war from the base ubuntu to the container OS’s liberty server’s dropin directory.<br/>
`docker cp ./sample.war 336c881a0ec6:/wlp/usr/servers/AppServer/dropins`

14. Login to the container again. 

15. Navigate to /wlp/usr/servers/AppServer/dropins
Execute `ls` command, you should see sample.war present. 

16. Navigate to /wlp/bin and execute the following command to start the server:<br/>
`./server start AppServer`<br/>
When successfully started you should see the following message in the console:<br/>
Server AppServer started with process ID 5267.

17. Exit and saved the container.

18. Open the browser and hit the following URL:
http:// 9.199.144.167:9080/sample/hello.jsp

Following output will be seen:

<img width="300" alt="image" src="https://user-images.githubusercontent.com/93929892/185535311-1a243716-ced6-44d7-ac43-ffe1f5eeb4d6.png">

**Exercise 6:**

The page in exercise 4 was not accessible because the port 9080 is restricted within the container. You need to map a host port to the container port. <br/>

1. Save the container using the docker commit command.<br/>

2. Stop the docker container using the following command:<br/>
`docker stop <container-id>`

3. Remove the container using the following command:<br/>
`docker ps -a
docker rm <container-id>`

4. restart the docker using the following command:<br/>
`docker run -idt -p 9080:9080 <image-name>`

5. login into the docker container using docker exec command.<br/>
`Docker exec -it <container-id> bash`

6.  Navigate to /wlp/bin and execute the following command to start the server:<br/>
`./server start AppServer`

7. Hit the URL is local browser:<br/>
http:// 9.199.144.167:9080/sample/hello.jsp

Output:
<img width="630" alt="image" src="https://user-images.githubusercontent.com/93929892/185537075-6916daba-0cf4-42de-8604-ac7d9d890c1a.png">

8. Exit the container.

**Exercise 7**

Now we are going to build the entire environment that we have done throughout exercise 1 to exercise 5 using a single Dockerfile.<br/>

1. Update the Dockerfile as follows:

```DOCKER
FROM centos
RUN yum install -y yum-utils
RUN yum install java-1.8.0-openjdk -y
RUN yum install vim -y
RUN yum install zip -y
COPY ./wlp-webProfile8-20.0.0.7.zip /wlp.zip
RUN unzip wlp.zip
RUN cd wlp/bin \
&& ./server create appserver
RUN rm ./wlp/usr/servers/appserver/server.xml
COPY ./server.xml /wlp/usr/servers/appserver/server.xml
COPY ./sample.war /wlp/usr/servers/appserver/dropins/sample.war
COPY ./execute.sh /execute.sh
RUN chmod 777 -R ./execute.sh
CMD ["./execute.sh"]
```

2. Create the execute.sh file the LabSession1 directory. The execute.sh content is as follow:
`
#!/bin/sh
./wlp/bin/server start appserver
/bin/bash`

3. Create the server.xml file in the LabSession1 directory. The server.xml content is as follows:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<server description="new server">
    <featureManager>
        <feature>webProfile-8.0</feature>
    </featureManager>
    <httpEndpoint id="defaultHttpEndpoint" host="*" httpPort="9080" httpsPort="9443" />
    <applicationManager autoExpand="true"/>
</server>
```

4. You should now have the following files in the LabSession1 directory.<br/>

server.xml, execute.sh, Dockerfile, sample.war, wlp-webProfile8-20.0.0.7.zip

5. Build the docker image. 

6. Run the container
 
`docker run -idt -p 9080:9080 <image-id>`

7. Check the status by executing docker ps.

8. Hit the URL in the browser:
http:// 9.199.144.167:9080/sample/hello.jsp

**Exercise 8**

Creating Docker volume to view liberty logs

1. Docker container files can be copied to host machine using the docker cp command.
`docker cp 336c881a0ec6:/wlp.zip ./wlp-webProfile8-19.0.0.9.zip`

However, this will not be feasible for frequent access of the files. Docker volumes will solve this problem.

2.  Create a docker volume using the following command:
`docker volume create libertylogs`

3. List the volumes
`docker volume ls`

<img width="656" alt="image" src="https://user-images.githubusercontent.com/93929892/185801251-ad75b65a-9559-49f8-9374-57b83167a8e3.png">

4. Inspect the docker volume

<img width="656" alt="image" src="https://user-images.githubusercontent.com/93929892/185801283-a0ccdded-55b2-496c-bf11-6e998c7db5ff.png">

4.  To remove docker volume
`docker volume rm libertylogs`

5. Start the container with a volume

`docker run -idt --mount source=libertylogs,target=/wlp/usr/servers/appserver/logs ec753d93981`

Navigate to the volume directory and list the files

<img width="649" alt="image" src="https://user-images.githubusercontent.com/93929892/185801313-60fabf8c-4ed1-412f-9a36-eddef86ad24d.png">



