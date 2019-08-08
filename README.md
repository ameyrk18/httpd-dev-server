# httpd-dev-server

1. This is role is created to install and configure developer web server. 
2. SELinux is set to enforcing and targeted.
3. Each developer is assigned with a directory where he can deploy his code
4. Each developer's content is accessed via non standard port 
5. This role has dependency on organisational httpd role (``https://github.com/ameyrk18/httpd.git``) amd the version is ``v1.4``. The dependency is defined in meta/main.yml. If your using this role you can remove the dependency and checkout from master version
6. Configures self signed certificates
7. Modify SElinux settings booleans for httpd dev server 
8. Allows Apache to listen on specific tcp port for each developer by enabling seport
9. Deploys html code
10. Configures vhosts 
11. Validates httpd configuration
12. Restarts httpd

# How to execute this role 

```
git clone https://github.com/ameyrk18/httpd-dev-server.git 
cd httpd-dev-server 
ansible-playbook -i inventory dev-playbook.yml --private-key ~/.ssh/<private-key>   --extra-vars "ansible_user=<ansible-user>"
```

# Variable controls 

CA certs: By default self signed certificates are configured. If you want to add valid signed certificates then you need to update your certs under ``https://github.com/ameyrk18/httpd.git`` base role ``certs/others`` directory. You need to either create a new tag and update ``meta/main.yml``. 

```---
   - name: Converge
     hosts: all
     become: yes
     gather_facts: yes
     vars:
       ssl_type: selfsigned
       target_env_cert: others
       httpd_ssl_servername: somedomain.com     
     roles:
       - ../httpd-dev-server                    
 ```       
 
Configure developer requirements: The developer requirements are managed under ``vars/main.yml `` by using dictionary object (key and value pairs). 
The below data set creates users (dev1, dev2 etc.), developer directories like (mango.com, apple.com etc.) to host contents and finally non standard ports (8081,8082 etc.) You can add as much as you can here. This data is also used to configure vhosts configuration file which is defined under templates. 
```bash
v_hosts:
     dev1:
       website: mango.com
       port: 8081
     dev2:
       website: apple.com
       port: 8082
     dev3:
       website: papaya.com
       port: 8083
     dev4:
       website: grapes.com
       port: 8084
```
 
SElinux configurations: sebooleans are enabled by the below variable in order to access read/write logs. seport is enabled from the above dictionary by picking up the ports.  


```httpd_sebooleans:
     - httpd_enable_cgi
     - httpd_unified
     - httpd_builtin_scripting
 ```
 
 # Access website 
``` 
https://<yourip>:8081 (mango.com)
https://<yourip>:8082 (apple.com)
https://<yourip>:8083 (papaya.com)
https://<yourip>:8084  (grapes.com)
```

# Molecule unit testing 

Prerequisite (tested on mac):

``brew install molecule ``

``pip install 'molecule[docker]'``

``git clone https://github.com/ameyrk18/httpd-dev-server.git ``

``molecule init``

``molecule test`` # run tests. sample test case will be executed and below will be the output.

```    ============================= test session starts ==============================
       platform darwin -- Python 2.7.16, pytest-4.6.5, py-1.8.0, pluggy-0.12.0
       rootdir: /Users/ameykamble/IdeaProjects/quickies/final-submission/httpd-dev-server/molecule/default
       plugins: testinfra-3.0.6, pylama-7.4.3
   collected 1 item                                                               
       
       tests/test_default.py .                                                  [100%]
       
       =========================== 1 passed in 3.61 seconds ===========================
```

TO DO: 

1. Add actual unit test cases 
2. Test the pipeline and get that working in jenkins
