# Privacyidea Kurulumu Microsoft RDP ve OpenVPN Entegrasyonu
Bu makalede, PrivacyIDEA ve OpenVPN kullanarak iki faktörlü kimlik doğrulama (2FA) sistemi kurulumunun adımlarını ele alıyoruz.

Linux dağıtımı olarak Ubuntu 24.04.1 LTS kullanılmıştır.

### PrivacyIDEA Kurulumu

#### Zaman Dilimi Ayarı

Zaman dilimini ayarlayın:
```bash
sudo timedatectl set-timezone Europe/Istanbul
```

#### Gerekli Paketlerin Yüklenmesi

1. Python ve Virtualenv kurulumu:
```bash
sudo apt install python3-pip
sudo pip install virtualenv --break-system-packages
```

2. PrivacyIDEA ortamını oluştur:
```bash
sudo virtualenv -p /usr/bin/python3 /opt/privacyidea
cd /opt/privacyidea
source bin/activate
```

3. PrivacyIDEA paketlerini yükleyin:
```bash
sudo pip install privacyidea --break-system-packages
sudo chown -R $(whoami) /opt/privacyidea/
```

4. GPG anahtarını indirip ekleyin:
```bash
wget https://lancelot.netknights.it/NetKnights-Release.asc
sudo mv NetKnights-Release.asc /etc/apt/trusted.gpg.d/
sudo add-apt-repository http://lancelot.netknights.it/community/noble/stable
sudo apt update
sudo apt install privacyidea-apache2
```
```bash
Not: Eğer add-apt-repository: command not found hatası alırsanız o zaman aşağıdaki paketleri yüklemeniz gerekmekte.
sudo apt-get install software-properties-common
```
#### Web Arayüz Ayarları

1. Admin kullanıcısı oluşturun:
```bash
sudo pi-manage admin add admin admin@localhost
```

2. Arayüz dilini ayarlamak için dosyayı düzenleyin:
```bash
sudo nano /opt/privacyidea/lib/python3.12/site-packages/privacyidea/webui/login.py
```

Sadece İngilizce dilini aktif edin:

Not: Yerel dildeki çeviri çok başarılı olmadığı için ingilizce tercih edilmiştir.

```python
# DEFAULT_LANGUAGE_LIST = ['en', 'de', 'nl', 'zh Hant', 'fr', 'es', 'tr', 'cs', 'it']  #:
DEFAULT_LANGUAGE_LIST = ['en']  #:
```

3. Apache sunucusunu yeniden başlatın:
```bash
sudo systemctl restart apache2
```

#### MySQL Veritabanı Kurulumu

1. MySQL üzerinde veritabanı oluşturun:
```sql
CREATE DATABASE rdp_users CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
USE rdp_users;
GRANT ALL PRIVILEGES ON rdp_users.* TO 'pi'@'localhost';
FLUSH PRIVILEGES;
```

2. Kullanıcı tablosunu ekleyin:
```sql
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(255) NOT NULL UNIQUE,
  firstname VARCHAR(255),
  lastname VARCHAR(255),
  email VARCHAR(255),
  mobile VARCHAR(20),
  description TEXT,
  enabled BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

3. MYSQL tarafında erişim için kullanacağımız kullanıcı bilgilerini aşağıdaki dosyadan alın.
```bash
sudo nano /etc/privacyidea/pi.cfg
```

#### PrivacyIDEA Web Arayüz Ayarları

Aşağıdaki görselleri takip ederek kurulumu tamamlayın.
1. Yeni bir SQL Resolver tanımlayın (Config > Users sekmesi).
   - Resolver Name: `mysql-resolver`
   - Driver: `mysql-pymysql`
   - Server: `127.0.0.1`
   - Port: `3306`
   - Database: `rdp_users`
   - User: `pi`
   - Password: `EnQaAwbrs2iP`
   - Mapping: `{ "username": "username", "surname": "lastname", "givenname": "firstname", "email": "email", "mobile": "mobile", "description": "description", "userid": "id" }`

![Image 1](images/webui/privacyidea-webui-001.png)

![Image 2](images/webui/privacyidea-webui-002.png)

2. Yeni bir realm oluşturun (Config > Realms sekmesi) ve bunu varsayılan olarak ayarlayın.

![Image 3](images/webui/privacyidea-webui-003.png)

![Image 4](images/webui/privacyidea-webui-004.png)

3. Users sekmesinden kullanıcıyı kaydedin.

![Image 5](images/webui/privacyidea-webui-005.png)

4.1 Tokens bölümüne gidin. Enroll token bölümünden TOTP'yi seçin.

4.2 Az önce oluşturduğunuz realmi seçin.

4.3 Az önce oluşturduğunuz realmi seçin.

4.4 MySQL'de oluşturduğunuz kullanıcı adını Kullanıcı Adı olarak girin.

4.5 Pin alanını boş bırakın.

4.6 Bir kapsayıcıyı, Token'ı bir kapsayıcıya ata'yı seçerek yapılandırabilirsiniz.

4.7 Token'ı Kaydet'e tıklayarak kaydedin ve kullanıcıya atayın.

4.8 ​​Not: Bağlandığınız RDP sunucusundaki kullanıcı adı, PrivacyIDEA'da tanımladığınız kullanıcı adıyla eşleşmelidir.

![Image 6](images/webui/privacyidea-webui-006.png)

![Image 7](images/webui/privacyidea-webui-007.png)

![Image 8](images/webui/privacyidea-webui-008.png)

5. Bunlar Sistem Yapılandırması'ndaki ayarlarımdır. İsterseniz ayarları kendi yapınıza göre değiştirebilirsiniz.

![Image 9](images/webui/privacyidea-webui-009.png)

6. Bunlar Privacyidea'da kullandığım policy ayarlarıdır.

![Image 10](images/webui/privacyidea-webui-010.png)

![Image 11](images/webui/privacyidea-webui-011.png)

![Image 12](images/webui/privacyidea-webui-012.png)

![Image 13](images/webui/privacyidea-webui-013.png)

![Image 14](images/webui/privacyidea-webui-014.png)

![Image 15](images/webui/privacyidea-webui-015.png)

7. Denetim sekmesinden OTP doğrulamasının başarılı olduğunu gözlemleyin.

![Image 16](images/webui/privacyidea-webui-016.png)

### PrivacyIDEA Crendtial Provider Kurulumu

1. Not: RDP yapacağınız sunucu üzerinde ki kullanıcı ile privacyidea üzerinde tanımladığınız kullanıcı adı aynı olmalıdır.

2. Not : Kurulum Windows 2022 üzerinde gerçekleştirilmiştir.

3. Aşağıdaki görselleri takip ederek kurulumu tamamlayın.

![Resim 1](images/privacyidea-credential-provider-1.png)

![Resim 2](images/privacyidea-credential-provider-2.png)

![Resim 3](images/privacyidea-credential-provider-3.png)

![Resim 4](images/privacyidea-credential-provider-4.png)

![Resim 5](images/privacyidea-credential-provider-5.png)

![Resim 6](images/privacyidea-credential-provider-6.png)

![Resim 7](images/privacyidea-credential-provider-7.png)

![Resim 8](images/privacyidea-credential-provider-8.png)

![Resim 9](images/privacyidea-credential-provider-9.png)

![Resim 10](images/privacyidea-credential-provider-10.png)


---

Bu adımları tamamladıktan sonra, sisteminiz PrivacyIDEA ve RDP ile iki faktörlü kimlik doğrulama desteğiyle daha güvenli hale gelecektir.

---

### OpenVPN Tarafında 2FA Kurulumu

#### OpenVPN Kurulumu

1. Zaman dilimini ayarlayın:
```bash
sudo timedatectl set-timezone Europe/Istanbul
```

2. OpenVPN kurulumu:
```bash
wget https://raw.githubusercontent.com/angristan/openvpn-install/refs/heads/master/openvpn-install.sh
chmod +x openvpn-install.sh
sudo ./openvpn-install.sh
```

3. Linux sunucu üzerinde yeni bir kullanıcı ekleyin:
```bash
sudo adduser gokhan
```

4. Oluşturduğunuz kullanıcıyı OpenVPN üzerinden bağlanabilmesi için ekleyin:
```bash
sudo ./openvpn-install.sh
- Clientname: `gokhan`
- Password: `Add a passwordless client`
```

5. Not: VPN yapacağınız sunucu üzerinde ki kullanıcı ile privacyidea üzerinde tanımladığınız kullanıcı adı aynı olmalıdır.

#### PrivacyIDEA ile PAM Entegrasyonu

1. PrivacyIDEA PAM modülünü kurun:
```bash
git clone https://github.com/privacyidea/privacyidea-pam
cd privacyidea-pam
wget https://github.com/nlohmann/json/releases/download/v3.11.3/json.hpp
sudo apt install make g++ libcurl4-openssl-dev libssl-dev libpam0g-dev
make
make install
sudo mkdir /lib64/security/
sudo mv pam_privacyidea.so /lib64/security
```

2. PAM dosyasını düzenleyin:
```bash
sudo nano /etc/pam.d/openvpn
```
Dosyanın içeriği:
```plaintext
auth    required                        pam_unix.so     try_first_pass
auth    [success=1 default=die]         /lib64/security/pam_privacyidea.so      url=https://domainadiniz.org prompt=privacyIDEA_Authentication
auth    requisite                       pam_deny.so
auth    required                        pam_permit.so
account sufficient                      pam_permit.so
session sufficient                      pam_permit.so
```

3. OpenVPN sunucu ayar dosyasını düzenleyin:
```bash
sudo nano /etc/openvpn/server.conf
```
Ayarlar:
```plaintext
verify-client-cert none
username-as-common-name
reneg-sec 0
plugin /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so "openvpn login USERNAME password PASSWORD 'please enter otp:' OTP"
script-security 3
```

4. OpenVPN sunucusunu yeniden başlatın:
```bash
sudo systemctl restart openvpn@server.service
```

5. OpenVPN istemci ayar dosyasına aşağıdaki satırları ekleyin:
```bash
auth-user-pass
static-challenge "privacyIDEA_Authentication" 1
setenv FRIENDLY_NAME "privacyIDEA_Authentication"
```

---

Bu adımları tamamladıktan sonra, sisteminiz PrivacyIDEA ve OpenVPN ile iki faktörlü kimlik doğrulama desteğiyle daha güvenli hale gelecektir.

---

Daha fazla yardıma ihtiyacınız olursa lütfen bana bildirin!
