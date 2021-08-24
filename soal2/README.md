# Jawaban no. 2
1. buat dan masuk ke folder untuk menyimpan file vagrant
```
mkdir /root/espin-python && cd/root/espin-python
```
2. clone https://github.com/lumbans/devops-la-challenges
```
git clone https://github.com/lumbans/devops-la-challenges
```
3. copy file app.py dan requirements.txt ke folder espin-python
```
cp devops-la-challenges/app.py /root/espin-python/ && cp devops-la-challenges/requirements.txt /root/espin-python/
```
4. buat ```wsgi.py``` untuk gunicorn
```
from app import app

if __name__ == "__main__":
    app.run()

```
5. buat ```app.service``` agar dijadikan systemctl
```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
WorkingDirectory=/home/vagrant/
ExecStart=gunicorn --workers 3 --bind 0.0.0.0:5000 -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```
6. tambahkan Vagrantfile (Vagrantfile menyesuaikan soal yang diberikan)
```
nano Vagrantfile
```
```
Vagrant.configure("2") do |config|
config.vm.box = "bento/ubuntu-20.04"
  config.vm.network "forwarded_port", guest: 5000, host: 5000

  config.vm.network "private_network", ip: "10.10.10.20"

  config.vm.define "trusty" do |node|
    node.vm.hostname = "trusty.local"
  end
  config.vm.provision "ansible_local" do |ansible|
    ansible.install = true
    ansible.playbook = "main.yml"
  end
config.vm.provider "virtualbox" do |vb|
vb.memory = "1024"
     vb.cpus = "2"
   end
config.vm.provision "shell", inline: <<-SHELL
     apt-get update

   SHELL
end


```
7. tambahkan script ansible main.yml
```
- hosts: trusty
  gather_facts: no
  become: yes
  tasks:
  - name: Copy app.py to Vagrant Server
    copy:
      src: /$PWD/app.py
      dest: /home/vagrant/app.py
  - name: Copy wsgi.py to Vagrant Server
    copy:
      src: /$PWD/wsgi.py
      dest: /home/vagrant/wsgi.py
  - name: Copy app.service to Vagrant Server
    copy:
      src: /$PWD/app.service
      dest: /etc/systemd/system/app.service
  - name: install pip3
    apt:
      name: python3-pip
      state: latest
  - name: install requirements.txt
    pip:
      requirements: /vagrant/requirements.txt
      executable: pip3
  - name: install gunicorn
    apt:
      name: gunicorn
  - name: run app.py
    shell: |
      systemctl start app && systemctl enable app

```
8. hasil curl dari host(10.10.10.1) ke vm vagrant(10.10.10.20) dengan port 5000 ```curl 10.10.10.20:5000```
![image](https://user-images.githubusercontent.com/51534984/130605423-fb9fda6c-9281-41af-9f50-af6f250618ee.png)
