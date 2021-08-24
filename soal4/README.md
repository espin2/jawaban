# Jawaban no. 4
1. grok parser dari log sample-1.log
```
%{QS}:\"%{UUID:Data.UUID}\",%{QS}:%{WORD:Data.rt},%{QS}:\"%{WORD:Data.port}\",%{QS}:\"%{IP:Data.ip}\",%{QS}:\"%{WORD:Data.appli}\",%{QS}:\"%{WORD:Data.versi}\",%{QS}:\"%{UNIXPATH:Data.path}"
```
2. hasil grok parsing melalui dashboard kibana
![image](https://user-images.githubusercontent.com/51534984/130599801-e5cf96f2-4a2a-498d-a31b-f844f51c8c1b.png)
3. melakukan grok parsing melalui command line dan menambahkan ```sample-1.conf``` di ```/etc/logstash/conf.d/```
```
nano /etc/logstash/conf.d/sample-1.conf
```
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
4. hasil grok parsing melalui command line
![image](https://user-images.githubusercontent.com/51534984/130600520-010731bf-9781-47da-be39-6e18eecf8f82.png)


