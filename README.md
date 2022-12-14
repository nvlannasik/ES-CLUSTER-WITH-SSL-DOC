# ELK Stack Documentation

Repository ini berisi tentang tutorial instalasi dan konfigurasi Clustering Elasticsearch

## Environment For ElasticSearch Cluster

**Cluster** : 1 Cluster

**Node** : 3 Node

**OS** : Ubuntu Server 20.04

**Elasticsearch Version** : 8.x Latest

![App Screenshot](/gambar/environment.jpg)

# Elasticsearch Installation

Dalam dokumentasi resmi yang di publish disini [[1]](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html). Elasticsearch menyediakan berbagai macam pilihan metode/packages untuk berbagai environment seperti `Windows .zip archive`,`deb`,`rpm`,`docker`, dan `Elastic Cloud on Kubernetes`. Dan untuk dokumentasi ini kita akan menggunakan packages `deb` untuk instalasi elasticsearch.

Elasticsearch menggunakan Signing Key (PGP key D88E42B4, available from https://pgp.mit.edu) dengan fingerprint:

`4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4`

Download dan install public signing key:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

Install terlebih dahulu apt-transport-https package on Debian sebelum lanjut:

```bash
sudo apt-get install apt-transport-https
```

Simpan repository yang di definisikan di `/etc/apt/sources.list.d/elastic-8.x.list`:

```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

Lakukan update repository dan install elasticsearch

```bash
sudo apt-get update && sudo apt-get install elasticsearch -y
```

# Konfigurasi Node Elasticsearch

Setelah melakukan instalasi selanjutnya masuk kedalam file konfigurasi elasticsearch

```bash
nano /etc/elasticsearch/elasticsearch.yml
```

Uncomment syntax dibawah ini untuk mendefinisikan cluster elasticsearch dan beri nama sesuai dengan kebutuhan anda

```bash
#cluster.name: my-application
```

![Cluster Screenshot](/gambar/cluster.JPG)

Uncomment syntax dibawah ini untuk mendefinisikan node elasticsearch dan beri nama sesuai dengan kebutuhan anda

```bash
#node.name: node-1
```

![Node Screenshot](/gambar/node.png)

Uncomment syntax dibawah ini untuk mendefinisikan network host sesuai dengan IP Server yang dipakai dan juga port yang akan digunakan

```bash
#network.host: 192.168.0.1
#http.port: 9200
```

![Node Screenshot](/gambar/ipport.png)

Uncomment syntax dibawah ini untuk mendefinisikan discovery dari setiap host untuk clustering (gunakan IP host dari setiap node elasticsearch) dan juga cluster initial master node (gunakan nama node dari setiap node elasticsearch)

```bash
#discovery.seed_hosts: ["host2", "host2"]
#cluster.initial_master_nodes: ["node-1", "node-2"]
```

![Discovery Screenshot](/gambar/discovery.png)

Tambahkan syntax dibawah ini untuk menggunakan license elasticsearch yang bersifat gratis

```bash
xpack.license.self_generated.type: basic
```

![License Screenshot](/gambar/license.png)

comment syntax dibawah ini agar tidak terjadi bentrok

```bash
#cluster.initial_master_nodes: ["node-1"]
```

![init-master-node Screenshot](/gambar/comment.png)

Setelah semua sudah di konfigurasi sesuai dengan kebutuhan, lakukan save dan keluar dari text editor

# Konfigurasi ca dan certificate

Selanjutnya, kita akan melakukan generate ca untuk node elasticsearch. Masuk kedalam direktori elasticsearch

```bash
cd /usr/share/elasticsearch/bin
```

Lalu jalankan syntax dibawah ini untuk generate ca

```bash
./elasticsearch-certutil ca
```

masukan password untuk ca yang akan di generate

![ca Screenshot](/gambar/ca.png)

setelah berhasil di generate, file ca akan tersimpan di direktori `/usr/share/elasticsearch/`

```bash
cd ..
ls
```

![ca Screenshot](/gambar/ca2.png)

Selanjutnya, kita akan melakukan generate certificate untuk setiap node elasticsearch. Masuk kembali kedalam folder bin

```bash
cd bin
```

Lalu jalankan syntax dibawah ini untuk generate certificate

```bash
./elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

masukan password yang sudah di buat sebelumnya untuk ca yang akan di generate

![cert Screenshot](/gambar/ca3.png)

cek file certificate yang sudah di generate

```bash
cd ..
ls
```

![cert Screenshot](/gambar/ca4.png)

# Konfigurasi HTTPS Certificate

Selanjutnya, kita akan melakukan generate https certificate untuk node elasticsearch. Masuk kembali kedalam folder bin

```bash
cd bin
```

Lalu jalankan syntax dibawah ini untuk generate https certificate

```bash
./elasticsearch-certutil http
```

pilih opsi `n` untuk pilihan generate CSR, karena kita akan menggunakan file yang sudah di generate sebelumnya. Lalu pilih opsi `y` untuk menggunakan file yang sudah di generate sebelumnya

![https Screenshot](/gambar/http1.png)

Masukan direktori path dari file ca yang sudah di generate sebelumnya

```bash
/usr/share/elasticsearch/elastic-stack-ca.p12
```

setelah itu masukan password ca yang sudah di buat sebelumnya
![https Screenshot](/gambar/http2.png)

atur berapa lama certificate akan berlaku. pencet `enter` untuk menggunakan default value `5y` (5 tahun)

![https Screenshot](/gambar/http3.png)

untuk production, sebaiknya pilih opsi `y` untuk generate 1 certificate untuk setiap node elasticsearch. Lalu masukan nama node elasticsearch yang akan di generate certificate nya

![https Screenshot](/gambar/http4.png)

![https Screenshot](/gambar/http5.png)

kemudian masukan setiap hostname dari setiap node elasticsearch
![https Screenshot](/gambar/http6.png)

masukan IP address dari setiap node elasticsearch

![https Screenshot](/gambar/http7.png)

setelah itu masukan password untuk certificate yang akan di generate

![https Screenshot](/gambar/http8.png)

hasil generate certificate akan tersimpan di direktori `/usr/share/elasticsearch/` dan file akan berbentuk zip

![https Screenshot](/gambar/http9.png)

ekstract file zip tersebut, dan akan terlihat 2 folder yaitu `elasticsearch` dan `kibana`. Yang dimana berisi file https certificate dan private key untuk kibana

![https Screenshot](/gambar/http10.png)

# Move File Certificate dan masukan dalam 1 Folder

Selanjutnya, kita akan memindahkan file certificate yang sudah di generate kedalam 1 folder. Buat folder didalam direktori elasticsearch dan pinahkan file certificate kedalam folder tersebut

```bash
mkdir /etc/elasticsearch/certs/cluster
mv /usr/share/elasticsearch/elastic-certificates.p12 /etc/elasticsearch/certs/cluster
mv /usr/share/elasticsearch/elasticsearch/http.p12 /etc/elasticsearch/certs/cluster
mv /usr/share/elasticsearch/kibana/elasticsearch-ca.pem /etc/elasticsearch/certs/cluster
mv /usr/share/elasticsearch/elastic-stack-ca.p12 /etc/elasticsearch/certs/cluster
cd /etc/elasticsearch/certs/cluster
ls
```

![move Screenshot](/gambar/move.png)

# Update Konfigurasi Elasticsearch dengan certificate yang sudah di generate

Edit kembali file konfigurasi elasticsearch.yml

```bash
nano /etc/elasticsearch/elasticsearch.yml
```

ubah direktori path dari file certificate yang sudah di generate sebelumnya

![config Screenshot](/gambar/updatecert.png)

# Update password keystores dan truststores

Selanjutnya, kita akan melakukan update password dari file keystore dan truststore yang sudah kita buat sebelumnya.

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
```

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password
```

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
```

```bash
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

Jalankan service elasticsearch dan `enabled` service agar otomatis berjalan saat server mengalami reboot

```bash
systemctl start elasticsearch
systemctl enable elasticsearch
```

# Reset Password Elasticsearch

Di tahap ini kita akan melakukan reset password elasticsearch untuk authentication yang dimana sudah di auto generate oleh elasticsearch. Untuk itu kita akan melakukan reset password, gunakan syntax dibawah ini

```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic -i
```

Masukan password yang diinginkan untuk user elastic

![reset Screenshot](/gambar/resetPass.png)

test elasticsearch dengan menggunakan `curl`

```bash
curl -XGET -u elastic:<password> https://<ip address>:9200
```

Maka akan di respon seperti dibawah ini

```
{
  "name" : "node-1",
  "cluster_name" : "es-cluster",
  "cluster_uuid" : "44WMGvySTcuIbRCDKr-Evg",
  "version" : {
    "number" : "8.5.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "4ed5ee9afac63de92ec98f404ccbed7d3ba9584e",
    "build_date" : "2022-12-05T18:22:22.226119656Z",
    "build_snapshot" : false,
    "lucene_version" : "9.4.2",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

# Distribusi Certificate ke setiap node

Selanjutnya, kita akan melakukan distribusi certificate `http.p12` dan `elastic-certificates` yang sudah di generate ke setiap node elasticsearch. Untuk itu kita akan melakukan copy file certificate yang sudah di generate ke setiap node elasticsearch.

```bash
scp /etc/elasticsearch/certs/cluster/elastic-certificates.p12 root@<ip address node>:/etc/elasticsearch/certs/cluster

scp /etc/elasticsearch/certs/cluster/http.p12 root@<ip address node>:/etc/elasticsearch/certs/cluster
```

Setelah proses copy selesai, silahkan ulangi langkah - langkah diatas untuk setiap node elasticsearch.

# Instalasi & Konfigurasi Kibana

Untuk kibana ini akan kita install pada `Node-1`

Instal kibana dengan menggunakan perintah dibawah ini

```bash
apt-get install kibana -y
```

buka file konfigurasi kibana dan lakukan konfigurasi sesuai dengan kebutuhan

```bash
nano /etc/kibana/kibana.yml
```

Uncomment syntax dibawah ini untuk mendefinisikan port dari kibana

```bash
#server.port: 5601
```

dan uncomment syntax dibawah ini untuk mendefinisikan network host sesuai dengan IP Server yang dipakai

```bash
#server.host: "localhost"
```

![Cluster Screenshot](./gambar/kibana1.png)

uncoment syntax dibawah ini dan masukan host setiap node di cluster elasticsearch

```bash
#elasticsearch.hosts: ["http://localhost:9200"]
```

![Cluster Screenshot](./gambar/kibana2.png)

uncoment syntax dibawah ini dan masukan username dan password dari user `kibana_system` yang dimana nanti akan kita reset passwordnya

```bash
#elasticsearch.username: "kibana_system"
#elasticsearch.password: "changeme"
```

![Cluster Screenshot](./gambar/kibana3.png)

uncoment syntax dibawah ini dan masukan path dari file certificate yang sudah di generate sebelumnya

```bash
#elasticsearch.ssl.certificateAuthorities: ["/path/to/your/CA.pem"]
```

![Cluster Screenshot](./gambar/kibana4.png)

setelah di konfigurasi sesuai dengan kebutuhan, lakukan save dan keluar dari text editor

# Reset Password Kibana

Di tahap ini kita akan melakukan reset password kibana_system untuk authentication yang dimana sudah di auto generate oleh elasticsearch. Untuk itu kita akan melakukan reset password, gunakan syntax dibawah ini

```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system -i
```

# Move PEM file ke folder Kibana

Setelah melakukan reset password, kita akan melakukan copy file `elasticsearch-ca.pem` ke folder kibana

```bash
cp /etc/elasticsearch/certs/cluster/elasticsearch-ca.pem /etc/kibana/
```

# Jalankan Service Kibana

Jalankan service kibana dengan perintah dibawah ini

```bash
systemctl start kibana
systemctl enable kibana
```

akses kibana di web browser dengan alamat `http://<ip host>:5601` dan masukan username dan password dari user `elastic` yang sudah kita buat sebelumnya

```bash
http://192.168.1.14:5601
```

![Kibana Screenshot](./gambar/kibana5.png)

instalasi cluster elasticsearch dan kibana sudah selesai
![Kibana Screenshot](./gambar/kibana6.png)

## Authors

- [Ahmad Naoval Annasik](https://www.github.com/octokatherine)
