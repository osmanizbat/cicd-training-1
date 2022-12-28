## Virtualbox kurulumu
https://www.virtualbox.org/wiki/Downloads adresinden uygun kurulum paketi indirilerek kurulum gerçekleştirilir.

## Sanal sunucu oluşturulması
- [debian-live-11.6.0-amd64-standard.iso](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-11.6.0-amd64-standard.iso) dosyası indirilerek Virtualbox üzerinde yeni bir sanal sunucuya oluşturulur (Name: Jenkins, Memory: 2048 MB, cpu: 1, Disk: 20GB).
- Açılış menüsünde VBoxUnatendedInstall seçilerek kurulum başlatılır.
- Kurulum sonrası makine kapatılarak Machine -> Settings ekranında network ayarlarında "Bridged Adaptor" seçilerek makine start edilir.
- VM'e ssh ile bağlanabilmek için aşağıdaki komutlar çalıştırılır:

~~~
su - root
apt update && apt install openssh-server -y
usermod -aG sudo vboxuser
ip a
~~~

- Yukarıdaki adımlar tekrar uygulanarak aynı ayarlarla app-server adıyla bir sanal sunucu daha oluşturulur.

~~~
echo <jenkins_ip_adresi> jenkins >> C:\Windows\System32\drivers\etc\hosts
echo <app-server_ip_adresi> app-server >> C:\Windows\System32\drivers\etc\hosts
~~~

- Windows ayarlarında Apps & Features / Optional Fetures ekranında __OpenSSH Client__ uygulamasının kurulu olduğu teyit edilir.
- Windows terminal ekranıonda ssh vboxuser@jenkins komutuyla VM'e bağlanılır.
- Tercihen sunuculara public key authentication ile girebilmek için Windows terminalde "ssh-keygen -t rsa -b 4096" komutuyla key oluşturularak .ssh/id_rsa.pub dosyasının içeriği hedef sunucuda .ssh/authorized_keys dosyası içerisine eklenir. 


## Jenkins kurulumu

- Paket depolarına erişim için gerekli key dosyası sisteme eklenir:
~~~
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
~~~

- Repository listesi sisteme eklenir:
~~~
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
~~~

- Paket indexi güncellenir ve kurulum gerçekleştirilir:
~~~
sudo apt-get update
sudo apt-get install fontconfig openjdk-17-jdk git maven vim rsync
sudo apt-get install jenkins
sudo echo <app-server_ip_adresi> app-server >> /etc/hosts
sudo -u jenkins ssh-keygen -t rsa -b 4096
~~~

- Web tarayıcıda http://jenkins:8080 açılarak Jenkins kurulum adımları tamamlanır.


## Uygulama sunucusunun hazırlanması


- Uygulama dizinlerini ve çalıştıracak kullanıcıyı oluşturuyoruz.
~~~
sudo useradd -d /opt/spring-petclinic spring-petclinic
~~~

- Jenkins sunucusundan /var/lib/jenkins/.ssh/id_rsa.pub dosyasının içeriği app-server sunucusuna kopyalanır. 
~~~
sudo -u spring-petclinic vim /opt/spring-petclinic/.ssh/authorized_keys
sudo chmod 600 /opt/spring-petclinic/.ssh/authorized_keys
~~~

- Systemd servisini oluşturup uygulamayı başlatıyoruz.
~~~
sudo vim /lib/systemd/system/spring-petclinic.service
cd /etc/systemd/system/multi-user.target.wants
sudo ln -s /lib/systemd/system/spring-petclinic.service spring-petclinic.service
sudo systemctl enable spring-petclinic.service
sudo systemctl start spring-petclinic.service
sudo systemctl status spring-petclinic.service
~~~


https://github.com/spring-projects/spring-petclinic

https://github.com/cihanca/devopz

https://github.com/dotnet-architecture/eShopOnContainers


