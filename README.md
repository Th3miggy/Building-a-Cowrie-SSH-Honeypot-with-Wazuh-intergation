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

Ref 4: Once that is complete we can start to setup Cowrie virtual environment and install the requirements

        $ cd ~/cowrie
        $ virtualenv --python=python3 cowrie-env
        $ source cowrie-env/bin/activate
        $ pip install --upgrade pip
        $ pip -r requirements.txt

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-28%2019-56-46.png?raw=true)

Ref 5: Now we will Configure Cowrie. Following key changes that were made Hostname = fake-server listen_endpoints = tcp:6:2222:interface=:: SFTP_enables = true JSON logging enabled at: /home/cowrie/cowrie/var/log/cowrie/cowrie.json

        $ cp etc/cowrie.cfg.dist etc/cowrie.cfg
        $ nano etc/cowrie.cfg

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2019-39-00.png?raw=true)

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2019-42-01.png?raw=true)

Ref 6: For the next step we will start Cowrie and verify its running and we can as confirm its broadcasting on the correct port

        $ bin/cowrie start
        $ bin/cowrie status

Notes: worth saying documentation states that the cowrie command should be located in the bin folder however, Mine was not there but after some searching it was found and since it can't but run is a where the .cfg files isnt so I did have to go though the trouble of moving it to the correct location in the /bin directory 

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2020-38-50.png?raw=true)

test the port

        $ nc -zv YOUR_SERVER_IP 2222

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2020-41-36.png?raw=true)

Ref 7: Now that the Cowrie is up and running its time to get our Wazuh agent installed and register to our manager (note: reminder that lab already had a Wazuh manager setup on a spread machine which is required for the monitoring. Make sure you switch to Root starting now.

        $ curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
        $ echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

        $ sudo apt update
        $ sudo apt install wazuh-agent -y

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2022-14-23.png?raw=true)

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2022-15-05.png?raw=true)

Ref 8: Before we check the Wazuh manager we need to edit the ossec.conf file to monitor cowrie logs

        $ edit /var/ossec/etc/ossec.conf

and add this localfile block

        $ <localfile>
          $ <location>/home/cowrie/cowrie/var/log/cowrie/cowrie.json</location>
          $ <log_format>json</log_format>
        $ </localfile>

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-28%2021-45-09.png?raw=true)

Ref 9: Alright lets restart our wazuh-agent and check it in wazuh manager

        $ sudo systemctl restart wazuh-agent

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-27%2022-58-56.png?raw=true)

As you can see we now have a active agent "honeypot-cowrie" ID 200 on Ubuntu 26.04 LTS

I let the honeypot sit open for about 12 hours and we had right at 400 hits/alerts go off several of them being fail logon attempts

![not-working](https://github.com/Th3miggy/Building-a-Cowrie-SSH-Honeypot-with-Wazuh-intergation/blob/main/Screenshot%20from%202026-07-28%2018-53-07.png?raw=true)







