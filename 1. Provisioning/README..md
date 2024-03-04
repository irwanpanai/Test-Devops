# Provisioning
```
Requirements
- ansible
- server:
    GCP(appserver) : 2CPU 4GB RAM
    Biznet(Gateway): 1CPU 1GB RAM
```
  
1. Melakukan Instalasi Docker pada kedua server melalui ansible

```
---
- become: true
  gather_facts: true
  hosts: all
  tasks:

    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg
        - lsb-release
    - name: Docker GPG Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
    - name: Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
    - name: Docker Engine
      apt:
        update-cache: true
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
          - docker-buildx-plugin
    - name: Start and enable Docker service
      service:
        name: docker
        state: started
        enabled: yes
    - name: add user docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes
      become: yes
```

2. melakukan install nginx di gateway beserta reverse proxy

```rproxy.conf```
```
server {
        listen 80;
        listen 443 ssl;
        server_name shop.irwan.studentdumbways.my.id;

        ssl_certificate /etc/letsencrypt/live/irwan.studentdumbways.my.id/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/irwan.studentdumbways.my.id/privkey.pem;

        location / {
                proxy_pass http://34.16.184.199:8888;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }
}
```

```
---
- become: true
  gather_facts: false
  hosts: gateway
  tasks:
    - name: "Installing nginx"
      apt:
        name: nginx
        state: latest
        update_cache: yes
    - name: start nginx
      service:
        name: nginx
        state: started
    - name: copy proxy.conf
      copy:
        src: /home/ubuntu/ansible/rproxy.conf
        dest: /etc/nginx/sites-enabled/

```

5. melakukan generate SSL certificate

buat file untuk credential dns cloudflare
```
# Cloudflare API credentials used by Certbot
dns_cloudflare_email= "irwanpanai@gmail.com"
dns_cloudflare_api_key= "679de6c50a1bb060577cb18244b569a38f9ad"
```

bua file ```ssl.yml```

```
---
- name: Obtain SSL certificate
  hosts: gateway
  become: true
  tasks:
    - name: Install Certbot using snap
      become_user: root
      shell: |
       sudo snap install --classic certbot &&
       sudo snap set certbot trust-plugin-with-root=ok &&
       sudo snap install certbot-dns-cloudflare
      register: certbot_install_result
      changed_when: "certbot_install_result.rc == 0"

    - name: Create Certbot configuration directory
      file:
          path: "/home/irwansyah/.secrets"
          state: directory

    - name: copy cloudflare.ini
      copy:
        src: /home/irwan/ansible/cloudflare.ini
        dest: /home/irwansyah/.secrets/
        mode: 0400

    - name: Run Certbot to obtain certificates
      command: >
        certbot certonly
        --dns-cloudflare
        --dns-cloudflare-credentials /home/irwansyah/.secrets/cloudflare.ini
        --agree-tos
        --non-interactive
        --email irwanpanai@gmail.com
        --domains *.irwan.studentdumbways.my.id
      become_user: root
```
