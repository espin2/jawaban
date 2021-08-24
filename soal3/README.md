# Jawaban no. 3
1. grok parser dari log sample-1.log
```
%{TIMESTAMP_ISO8601:date}\|%{WORD:serviceid}\{\"channelSessionId\":\"%{WORD:channelSessionId}\",\"remark\":\"%{WORD:remark}\",\"OriginatorConversationID\":\"%{WORD:OriginatorConversationID}\",\"Msisdn\":\"%{WORD:Msisdn}\",\"CommandId\":\"%{WORD:CommandId}"
```
2. hasil grok parsing melalui dashboard kibana
![image](https://user-images.githubusercontent.com/51534984/130596725-7b86b2bc-b9bb-4b8b-8dd4-88a0b81c9e3d.png)
3. hasil grok parsing melalui https://grokdebug.herokuapp.com/
![image](https://user-images.githubusercontent.com/51534984/130596811-30873e8e-771e-4071-a4f8-22518be3807d.png)
4. melakukan grok parsing melalui command line dan menambahkan ```sample-1.conf``` di ```/etc/logstash/conf.d/```
```
nano /etc/logstash/conf.d/sample-1.conf
```
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
5. hasil grok parsing melalui command line
![image](https://user-images.githubusercontent.com/51534984/130599006-4093de7d-59d2-4edc-8cb3-f803fb308e5c.png)
