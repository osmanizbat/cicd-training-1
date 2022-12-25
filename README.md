## Virtualbox kurulumu
https://www.virtualbox.org/wiki/Downloads adresinden uygun kurulum paketi indirilerek kurulum gerçekleştirilir.

## Sanal sunucu oluşturulması
[debian-live-11.6.0-amd64-standard.iso](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-11.6.0-amd64-standard.iso) dosyası indirilerek Virtualbox üzerinde yeni bir sanal sunucu oluşturulur (Memory: 2048 MB, cpu: 1, Disk: 20GB).


## Jenkins kurulumu

~~~
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
~~~

~~~
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
~~~

~~~
sudo apt-get update
sudo apt-get install fontconfig openjdk-11-jre
sudo apt-get install jenkins
~~~