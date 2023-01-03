## Virtualbox kurulumu
https://www.virtualbox.org/wiki/Downloads adresinden uygun kurulum paketi indirilerek kurulum gerçekleştirilir.

## Sanal sunucu oluşturulması
1. [debian-live-11.6.0-amd64-standard.iso](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-11.6.0-amd64-standard.iso) dosyası indirilerek Virtualbox üzerinde yeni bir sanal sunucu oluşturulur (Name: Jenkins, Memory: 2048 MB, cpu: 1, Disk: 20GB).
2. Açılış menüsünde __VBoxUnatendedInstall__ seçilerek kurulum başlatılır (Açılış menüsü hızlı geçtiğinden açılış esnasında klavye ok tuşlarıyla menüyü yakalayabilirsiniz).
3. Kurulum sonrası makine kapatılarak Machine -> Settings ekranında network ayarlarında __"Bridged Adaptor"__ seçilerek makine start edilir.
4. VM'e ssh ile bağlanabilmek için aşağıdaki komutlar çalıştırılır:

    ~~~
    su - root
    apt update
    apt install openssh-server -y
    usermod -aG sudo vboxuser
    reboot
    ~~~

5. Yukarıdaki adımlar tekrar uygulanarak aynı ayarlarla __app-server__ adıyla bir sanal sunucu daha oluşturulur.

Yukarıda __vboxuser__ kullanıcısını __sudo__ grubuna eklediğimiz için bundan sonra root yetkisi gerektiren komutları önüne sudo ekleyerek çalıştırabiliriz.

6. Her iki sunucuda da aşağıdaki komut çalıştırılarak makine IP adresleri not edilir:
    ~~~
    ip a
    ~~~

7. Makinelere hostname ile erişim sağlayabilmek için Windows mekinenizde __Command Prompt__ uygulamasını administrator olarak açtıktan sonra aşağıdaki komutlarla Windows hosts dosyanıza IP adresleri eklenir.
    ~~~
    echo <jenkins_ip_adresi> jenkins >> C:\Windows\System32\drivers\etc\hosts
    echo <app-server_ip_adresi> app-server >> C:\Windows\System32\drivers\etc\hosts
    ~~~

8. Windows ayarlarında Apps & Features / Optional Fetures ekranında __OpenSSH Client__ uygulamasının kurulu olduğu teyit edilir.
9. Windows terminal ekranıonda `ssh vboxuser@jenkins` komutuyla VM'e bağlanılır.
10. Tercihen sunuculara public key authentication ile girebilmek için Windows terminalde `ssh-keygen -t rsa -b 4096` komutuyla key oluşturularak .ssh/id_rsa.pub dosyasının içeriği hedef sunucuda .ssh/authorized_keys dosyası içerisine eklenir. 


## Jenkins kurulumu

1. Jenkins paket depolarına erişim için gerekli key dosyası sisteme eklenir:
    ~~~
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
        /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    ~~~

2. Repository listesi sisteme eklenir:
    ~~~
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
        https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
        /etc/apt/sources.list.d/jenkins.list > /dev/null
    ~~~

3. Paket indexi güncellenir ve kurulum gerçekleştirilir:
    ~~~
    sudo apt-get update
    sudo apt-get install fontconfig openjdk-17-jdk git maven
    sudo apt-get install jenkins
    sudo echo "<app-server_ip_adresi> app-server" >> /etc/hosts
    sudo -u jenkins ssh-keygen -t rsa -b 4096
    ~~~

4. Web tarayıcıda http://jenkins:8080 açılarak Jenkins kurulum adımları tamamlanır.


## Uygulama sunucusunun hazırlanması

1. Paket indexi güncellenir ve gerekli bazı paketler kurulur
    ~~~
    sudo apt-get update
    sudo apt-get install openjdk-17-jdk acl
    sudo echo "<app-server_ip_adresi> app-server" >> /etc/hosts
    sudo -u jenkins ssh-keygen -t rsa -b 4096
    ~~~

2. Uygulama dizinini ve çalıştıracak kullanıcıyı oluşturuyoruz.
    ~~~
    sudo useradd -m -d /opt/spring-petclinic spring-petclinic
    sudo passwd spring-petclinic
    ~~~

3. Deployment kullanıcısını oluşturuyoruz. 
    ~~~
    sudo useradd -m jenkins
    sudo passwd jenkins
    ~~~

4. jenkins user için uygulama dizinine yazma yetkisi veriyoruz. 
    ~~~
    sudo setfacl -m u:jenkins:wx /opt/spring-petclinic
    ~~~

5. Jenkins sunucusundan jenkins user ssh key'ini app-server sunucusuna kopyalıyoruz. 
    ~~~
    sudo su - jenkins
    ssh-copy-id jenkins@app-server
    ~~~

6. Systemd servisini oluşturup uygulamayı başlatıyoruz.
    ~~~
    sudo vim /lib/systemd/system/spring-petclinic.service
    cd /etc/systemd/system/multi-user.target.wants
    sudo ln -s /lib/systemd/system/spring-petclinic.service spring-petclinic.service
    sudo systemctl enable spring-petclinic.service
    ~~~

7. jenkins user ile servisi restart edebilmek için aşağıdaki 2 satırı içeren sudoer dosyası oluşturuyoruz    
    ~~~
    sudo visudo -f /etc/sudoers.d/spring-petclinic 
    ~~~

    ~~~
    Cmnd_Alias COMMANDS = /usr/bin/systemctl restart spring-petclinic.service
    jenkins ALL = (root) NOPASSWD: COMMANDS
    ~~~

8. http://jenkins:8080 adresinden Jenkins'in web arayüzüne girerek pipeline oluşturarak "Pipeline script from SCM / Repository URL" kısmına bu git repository'nin adresi yazılır (https://github.com/osmanizbat/cicd-training-1)

9. Pipeline çalıştırıldıktan sonra app-server'da aşağıdaki komutla servisin çalışıp çalışmadığı kontrol edilir.  
    ~~~
    sudo systemctl status spring-petclinic.service
    ~~~

10. Web tarayıcıda http://app-server:8080 adresi açılarak uygulamaya erişilir.


