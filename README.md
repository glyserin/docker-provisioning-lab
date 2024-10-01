### Usage

**Environment: Mac M1, VMWare Fusion**

- Vagrant VMware Utility must be installed.
    → https://developer.hashicorp.com/vagrant/install/vmware
    - Only AMD is here... Install roetta to run in ARM  
        → `$ /usr/sbin/softwareupdate --install-rosetta --agree-to-license`
      <img src="https://github.com/user-attachments/assets/4a29bb91-1691-42bd-9cf1-796c4173649c" width="80%"/>
      <img src="https://github.com/user-attachments/assets/0bc05c67-e219-47f9-97a3-b513293a70b8" width="80%"/>

- Install vagrant  
    → `$ brew install --cask vagrant`  
    
- Next, install the Vagrant VMware provider plugin using the standard plugin installation procedure:  
    → `$ vagrant plugin install vagrant-vmware-desktop`  
    
- Update Vagrant to the latest version of the Vagrant VMware Desktop plugin:  
    → `$ vagrant plugin update vagrant-vmware-desktop`  
    
- Installing box  
    → `$ vagrant box add gyptazy/ubuntu22.04-arm64`  
    from: https://app.vagrantup.com/gyptazy/boxes/ubuntu22.04-arm64  
  <img src="https://github.com/user-attachments/assets/5ab39d63-3a1e-4623-9dcf-1ee0e694a4ff" width="80%"/>
  <img src="https://github.com/user-attachments/assets/dd16af50-56f9-4f64-a388-18632479fca4" width="80%"/>

- `host# vagrant up` → Create VM. Do this where Vagrantfile is located  
- `host# vagrant down` → Delete VM  
- `host# vagrant destroy` → Delete VM and its image  
- `host# vagrant ssh` → Enter the VM
- `vm$ ansible --version` → Check the version

<br>

**Register host in Ansible**
- `vm$ sudo bash`  
- `root@vm$ echo "localhost" >> /etc/ansible/hosts`  
- `root@vm$ exit`  

**Start NGINX Service**
- Install  
    - `vm$ sudo apt install -y nginx`  
- Start nginx with Ansible  
    - `vm$ ansible localhost -b -c local -m service -a "name=nginx state=started”`  


**Use `ansible-playbook`**
- https://github.com/devops-book/ansible-playbook-sample.git
- Get a playbook sample from Git  
    - `vm$ sudo apt install -y git`  
    - `vm$ git clone https://github.com/devops-book/ansible-playbook-sample.git`  

- `roles/common/tasks/main.yml`
    - Currently main.yml is declared to use yum package manager  
    - Comment this
    - Change here to use apt
    
    ```yaml
    # tasks file for common
    # - name: install epel
    #   yum: name=epel-release state=installed
    
    # tasks file for nginx
    - name: install nginx
      apt: name=nginx state=present
    
    - name: replace index.html
      template: src=index.html.j2 dest=/var/www/html/index.html  # 경로: centos 기준...
    
    - name: nginx start
      yum: name=nginx state=started enabled=yes
    ```

- Run the Playbook  
    `vm$ cd ansible-playbook-sample`  

- Move to Cloned ansible-playbook-sample directory  
    `vm$ ansible-playbook -i development site.yml`  
    → Inventory file is development, site.yml is playbook
  <img src="https://github.com/user-attachments/assets/1c6de9ff-527b-4aae-845b-a25e629b4901" width="80%"/>  

- Check the applied result  
    `vm$ curl localhost`  



### Creating a Docker-Installing Playbook**  
- At the directory where Vagrantfile is located, create a `playbook` directory  
- Create a Docker install role directory   
    - Ex: `playbook/roles/docker-install/`  
- Create a Docker install tasks directory 생성  
    - Ex: `playbook/roles/docker-install/tasks`  
- Define Task  
    - Ex: `playbook/roles/docker-install/tasks/main.yml`  
        - Add Docker GPG Key (APT)  
        - Add Docker Repository  
        - Install Docker CE  
        
        ```yaml
        - name: Add Docker GPG apt Key
          apt_key: 
            url: https://download.docker.com/linux/ubuntu/gpg 
            state: present
        
        - name: Docker Repository
          apt_repository: 
            repo: deb https://download.docker.com/linux/ubuntu focal stable
            state: present
        
        - name: Update apt and install docker-ce
          apt:
            name: docker-ce
            state: latest
            update_cache: true 
        ```

- Installing Docker CE  
    - Also compose the Docker Registry Pull Through Cache  
    - Expand Ansible plugin   
        `vm$ sudo ansible-galaxy collection install community.docker`
        - Check: `vm$ sudo ansible-galaxy collection list`
    - `community.docker.*` rules  
        - Image Pull rules (Docker pull)  
            ```yaml
            community.docker.docker_image:
              name: <image name>
              source: pull
            ```
        - Image container creating rules (Docker run)  
            ```yaml
            community.docker.docker_container:
              name: <container name>
              image: <image name>
              command: <command when starting container>
              mounts: <info to bind when starting container bind>
              state: present 
            ```

- Create new role  
    - Create Docker Registry Role directory  
        - Ex: `playbook/roles/docker-registry/`  
    - Define Task    
        - Ex: `playbook/roles/docker-registry/tasks/main.yml`  
        Compose new `main.yml` contents  
        ```yaml
        - name: Pull Docker Registry image
          community.docker.docker_image:
            name: registry:2
            source: pull
        
        - name: Create directory for registry config.yml
          file:
            path: /etc/docker/registry/
            state: directory
        
        - name: Configure registry config.yml
          copy:
            dest: /etc/docker/registry/config.yml
            content: |
              proxy:
                remoteurl: https://registry-1.docker.io
        
        - name: Create docker registry container
          community.docker.docker_container:
            name: registry
            image: registry:2
            detach: true
            recreate: true
            restart_policy: unless-stopped
            mounts:
            - type: bind
              source: /etc/docker/registry/
              target: /etc/registry/
            ports:
            - 5000:5000
            state: started
        
        - name: Update daemon.json
          copy:
            dest: /etc/docker/daemon.json
            content: |
              {
                "registry-mirrors": ["http://localhost:5000"]
              }
        
        - name: Restart docker daemon
          systemd:
            name: docker
            state: restarted
        ```


