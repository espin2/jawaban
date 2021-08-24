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
