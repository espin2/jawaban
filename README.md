# Jawaban no. 1
1. konfigurasi logrotate
```
nano /etc/logrotate.d/nginx
```
```
/var/log/nginx/*.log {
    size 100M
    weekly
    compress
    rotate 4
    missingok
    create 0664 root root
}

```
2. testing membuat file log lebih besar dari 100MB
```
$ fallocate -l 101M espin.log
$ fallocate -l 101M test.log
```
3. menjalankan logrotate secara manual untuk testing
```
logrotate /etc/logrotate.d/nginx
```
4. hasil setelah di logrorate, log akan di reset kembali dan log sebelumnya di compress menjadi .gz
![image](https://user-images.githubusercontent.com/51534984/130612086-dd86de6c-8c61-4655-a745-a9be0d5c4713.png)

5. masuk ke crontab -e
```
crontab -e
```
6. tambahkan script untuk menghapus log yang berada di /var/log/nginx yang sudah 7 hari. dan setiap 7 hari log akan dihapus
```
@weekly find /var/log/nginx/*.log -mtime +7 -type f -exec rm -rf {} \;
```
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

# Jawaban no. 3
```
%{TIMESTAMP_ISO8601:date}\|%{WORD:serviceid}\{\"channelSessionId\":\"%{WORD:channelSessionId}\",\"remark\":\"%{WORD:remark}\",\"OriginatorConversationID\":\"%{WORD:OriginatorConversationID}\",\"Msisdn\":\"%{WORD:Msisdn}\",\"CommandId\":\"%{WORD:CommandId}"
```
via dashboard kibana
![image](https://user-images.githubusercontent.com/51534984/130596725-7b86b2bc-b9bb-4b8b-8dd4-88a0b81c9e3d.png)
via https://grokdebug.herokuapp.com/
![image](https://user-images.githubusercontent.com/51534984/130596811-30873e8e-771e-4071-a4f8-22518be3807d.png)
```
input {
  file {
    path => "/root/sample-1.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
   grok {
        match => { "message" => "%{TIMESTAMP_ISO8601:date}\|%{WORD:serviceid}\{\"channelSessionId\":\"%{WORD:channelSessionId}\",\"remark\":\"%{WORD:remark}\",\"OriginatorConversationID\":\"%{WORD:OriginatorC$   }
}

output {
  elasticsearch {
     hosts => "localhost:9200"
     index => "demo-grok"
}
  stdout {}
}



```
via logstash cli
![image](https://user-images.githubusercontent.com/51534984/130599006-4093de7d-59d2-4edc-8cb3-f803fb308e5c.png)


# Jawaban no. 4
```
%{QS}:\"%{UUID:Data.UUID}\",%{QS}:%{WORD:Data.rt},%{QS}:\"%{WORD:Data.port}\",%{QS}:\"%{IP:Data.ip}\",%{QS}:\"%{WORD:Data.appli}\",%{QS}:\"%{WORD:Data.versi}\",%{QS}:\"%{UNIXPATH:Data.path}"
```
via dashboard kibana
![image](https://user-images.githubusercontent.com/51534984/130599801-e5cf96f2-4a2a-498d-a31b-f844f51c8c1b.png)
```
input {
  file {
    path => "/root/sample-2.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
   grok {
        match => { "message" => "{%{QS}:\"%{UUID:Data.UUID}\",%{QS}:%{WORD:Data.rt},%{QS}:\"%{WORD:Data.port}\",%{QS}:\"%{IP:Data.ip}\",%{QS}:\"%{WORD:Data.appli}\",%{QS}:\"%{WORD:Data.versi}\",%{QS}:\"%{UNIX$
   }
}

output {
  elasticsearch {
     hosts => "localhost:9200"
     index => "demo-grok"
}
  stdout {}
}

```
via logstash cli
![image](https://user-images.githubusercontent.com/51534984/130600520-010731bf-9781-47da-be39-6e18eecf8f82.png)


