# CI CD Demo

Untuk mengikuti latihan ini, Anda bisa menggunakan sistem operasi Windows, Linux, ataupun macOS. Namun, pastikan beberapa tools berikut sudah terinstal di komputer Anda.
- [Git] (https://git-scm.com/downloads) 
- [Docker] (https://www.docker.com/get-started/)
- Visual Studio Code (https://code.visualstudio.com/)

Jika semua tools tersebut sudah terpasang, simak tahapan proses yang akan kita lakukan di latihan berikut ini.

1. Dimulai dari menjalankan Jenkins melalui Docker.
2. Kemudian, menyiapkan Jenkins wizard.
3. Setelah itu, melakukan fork dan clone repository yang akan digunakan pada latihan ini.
4. Lalu, membuat Pipeline project di Jenkins.
5. Selanjutnya, menulis berkas Jenkinsfile untuk membuat build stage dan test stage di Jenkins pipeline.

## Menjalankan Jenkins di Docker
Di latihan ini, Anda akan menjalankan Jenkins sebagai Docker container dari jenkins/jenkins Docker image. Akan tetapi, karena image tersebut tidak bundle dengan Blue Ocean plugins (UX baru untuk Jenkins), kita juga akan menjalankan Blue Ocean sebagai Docker container. Silakan ikuti instruksi sesuai dengan sistem operasi yang Anda pakai.

1. Buka aplikasi Terminal pada komputer Anda.
2. Buatlah sebuah bridge network di Docker menggunakan perintah berikut.
``` 
docker network create jenkins 
```
3. Kita akan menjalankan aplikasi React App menggunakan Docker container di dalam sebuah Docker container (lebih tepatnya di dalam Blue Ocean container–nanti akan dibahas). Praktik ini disebut dengan dind alias docker in docker. Jadi, silakan unduh dan jalankan docker:dind Docker image menggunakan perintah berikut.
```
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
 ```
 4. Oke, sampai di sini, Docker container untuk Jenkins sudah berjalan. Namun, tugas kita belum selesai. Kita harus menjalankan Docker container lagi, kali ini untuk Blue Ocean (UX terbaru dari Jenkins). Untuk itu, buatlah sebuah Dockerfile dengan perintah berikut.
 5. Salin konten berikut ini ke Dockerfile Anda
 ```
 FROM jenkins/jenkins:2.346.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"
```
6. Simpan berkas tersebut dengan menekan CTRL+O, Enter, lalu CTRL+X
7. Buat sebuah Docker image baru dari Dockerfile tadi dan berikan nama myjenkins-blueocean:2.346.1-1
```
docker build -t myjenkins-blueocean:2.346.1-1 .
```
8. Setelah itu, jalankan myjenkins-blueocean:2.346.1-1 image sebagai container di Docker menggunakan perintah berikut.
```
docker run \
  --name jenkins-blueocean \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --volume "$HOME":/home \
  --restart=on-failure \
  --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
  myjenkins-blueocean:2.346.1-1 
 ```
 ## Menyiapkan Jenkins Wizard
 
Sebelum bisa mengakses Jenkins, ada beberapa langkah yang harus kita lakukan terlebih dahulu. Saat pertama kali mengakses Jenkins instance baru, kita diminta untuk unlock menggunakan password yang dibuat secara otomatis.

1. Buka browser Anda dan jalankan http://localhost:8080. Tunggu hingga halaman Unlock Jenkins muncul.
2. Sebagaimana yang tertulis di sana, Anda diminta untuk menyalin password dari Jenkins log untuk memastikan bahwa Jenkins diakses dengan aman oleh administrator.
3. Tampilkan Jenkins console log dengan perintah berikut di Terminal/CMD
```
docker logs jenkins-blueocean
```
4. Dari aplikasi Terminal/CMD, salin password yang ada di antara 2 rangkaian asterisk
5. Kembali ke halaman Unlock Jenkins di browser, paste password tersebut ke kolom Administrator password dan klik Continue.
6. Setelah itu, halaman Customize Jenkins muncul. Pilih Install suggested plugins. Setup wizard menunjukkan progres bahwa Jenkins sedang dikonfigurasi dan plugin yang disarankan sedang diinstal. Proses ini mungkin dapat memakan waktu hingga beberapa menit.
Catatan: Jika terdapat kegagalan saat proses instalasi plugin, Anda bisa klik Retry (untuk mengulangi proses instal plugin hingga berhasil) atau klik Continue (untuk melewati dan langsung melanjutkan ke langkah berikutnya).
7. Setelah beres, Jenkins akan meminta Anda untuk membuat administrator user. Saat halaman Create First Admin User muncul, isilah sesuai keinginan Anda dan klik Save and Continue.
8. Pada halaman Instance Configuration, pilih Save and Finish. Itu artinya, kita akan mengakses Jenkins dari url http://localhost:8080/. 
9. Saat halaman Jenkins is ready muncul, klik tombol Start using Jenkins.
Catatan: Halaman ini bisa jadi menunjukkan Jenkins is almost ready! Bila demikian, klik Restart. Jika halaman tersebut tidak refresh otomatis setelah beberapa menit, klik ikon refresh pada browser Anda secara manual.
10. Jika perlu, Anda bisa log in ulang ke Jenkins menggunakan kredensial yang tadi dibuat.


 
