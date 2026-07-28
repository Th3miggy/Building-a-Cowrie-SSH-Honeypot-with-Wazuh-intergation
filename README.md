# Building a Cowrie SSH Honeypot with Wazuh inegration

## Objective
[Brief Objective - Remove this afterwards]

Our objective in this lab will be to setup a Ubuntu server VM and install a Cowrie SSH Honeypot then connect the VM to our preexisting Wazuh manager and using the threat hunting tab capture real brute force attacks the server (Note: while this lab is not intended to be used a walk though if you do use it that way please make sure this honeypot is deploy in a isolated environment and never run it on a production system.) 

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Ubuntu Server VM running ubuntu 26.04 LTS 

## Steps

Ref 1: to Start this project off the first thing we need to do is to get our Ubuntu server Vm created and up and running. In my case I created the VM in a Proxmox environment a seen below 

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-28%2019-22-58.png?raw=true)

Ref 2: Once the vm is up and running our ubuntu server we will created a new user as it is best practice to run Cowrie under a non-root.

you can do that will the following command:

    $ sudo adduser --disabled-password cowrie

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2017-30-48.png?raw=true)

Ref 3: Now that the new user is created we go install our dependencies and clone Cowrie

        $ sudo apt update
        $ sudo apt install git python3-virtualenv libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind -y

Now switch to to the cowrie user and clone the repository

        $ sudo su - cowrie
        $ git clone https://github.com/cowrie/cowrie.git ~/cowrie

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2018-05-01.png?raw=true)




