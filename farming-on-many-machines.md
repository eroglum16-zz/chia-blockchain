Bunun için alternatif başlık:

# Ana makineniz haricindeki makinelerde nasıl hasat yaparsınız

Bu rehber her bir makinede tam bir düğüm, cüzdan veya çiftçi çalıştırmak zorunda kalmadan her bir makinede hasatçı çalıştırmanızı sağlar.
Bu sisteminizi daha basit tutar, daha az internet, depolama ve CPU kullanır, ayrıca anahtarlarınızın da daha güvenli kalmasını sağlar.
Bu durum genel olarak hasadınızı da daha hızlı ve zorluklara tepki vermede verimli yapar.

Mimari çiftçiyi tam düğüm ve cüzdanlı bir şekilde çalıştıran bir ana makineden ve sadece hasatçıyı çalıştıran diğer makinelerden meydana gelir.
Sadece ana makineniz Chia ağına bağlanacaktır.

Hasatçınız ve ana makineniz arasındaki iletişimi güvenli hale getirmek için,
ana makinenizin tüm sertifikaları imzalayan özel Sertifika Yetkilisi (SY) olacağı TLS kullanılır.
Ana makinenizle düzgün bir şekilde iletişim kurabilmesi için her bir hasatçının kendi imzalı sertifikası olmalıdır.

```                                          
                                       _____  Hasatçi 1 (sertifika A)
                                      /
diğer ağ çiftleri  --------   Ana makine (SY) ------  Hasatçı 2 (sertifika B)
                                      \_____  Hasatçı 3 (sertifika C)
```
## Ön Gereklilikler
* İlk olarak, Chia'nın tüm makinelerde yüklü ve `chia init` komutu ile başlatılmış olduğundan emin olun.
* Diğer hasatçılarda plot oluştururken `chia plots create -f çiftçi_anahtarı -p havuz_anahtarı` komutunu çalıştırın, 
havuz anahtarı ve çiftçi anahtarı kısımlarını ana makinenizden alarak doldurun.
Alternatif olarak, özel nahtarlarınızı `chia keys add` komutunu kullanarak da taşıyabilirsiniz, ancak bu diğeri kadar güvenli değildir.
Plot oluşturduktan sonra herşeyin düzgün çalıştığından emin olmak için `chia plots check` komutunu çalıştırın.
* SY klasörü ana makinenizde `~/.chia/mainnet/config/ssl/ca` dizini altındadır, 
bunun diğer hasatçılar tarafından erişilebilir olması için bir kopyasını alın;
`ssl/ca` klasörünü bir ağ sürücüsünde veya USB bellekte paylaşabilir, veya diğer hasatçılara ağ üzerinden gönderebilirsiniz. 
* `ssl/ca` klasörünü `chia-blockchain` 'in her sürümünde kopyalamanız gerekir,
bu sebeple eğer `beta` sürümünden `mainnet` sürümüne geçiyorsanız yeni `ca` klasörünün içindekileri kopyalamalısınız.

## Kurulum Adımları
Daha sonra tüm hasatçılar için aşağıdaki adımları izleyin:

**NOT:** 4. adım için, `/ca` klasörünün bir kopyasını geçici olarak ana makinenizden kullanıyorsunuz.
Hasatçıya bu dosyaları sadece bir kez gösteriyorsunuz, daha sonra bunları silebilirsiniz.

1. Ana makinenizin IP'sinin 8447 port'u üzerinden hasatçılarınız tarafından erişilebilir olduğundan emin olun. 
2. Tüm chia daemon işlemlerini `chia stop all -d` komutu ile sonlandırın.
3. Hasatçınızda herhangi bir ayar varsa yedeğini alın.
4. Hasatçınızda `chia init -c [klasör]` komutunu çalıştırın, `[klasör]` yazan yere ana makinenizdeki SY klasörünün kopyası gelmelidir.
Bu komut ana makinenizin SY'si tarafından imzalanmış yeni bir sertifika oluşturur.
5. Her bir hasatçıda `~/.chia/mainnet/config/config.yaml` dosyasını açın,
ve **`harvester`** bölümündeki farmer_peer kısmındaki HOST alanına ana makinenizin IP'sini girin.

ÖRN:
``` 
harvester:
  chia_ssl_ca:
    crt: config/ssl/ca/chia_ca.crt
    key: config/ssl/ca/chia_ca.key
  farmer_peer:
    host: Ana.Makine.IP
    port: 8447
```
6. `chia start harvester -r` komutu ile hasatçıyı başlatın, 
sonrasınd ana makinenizdeki INFO seviyesindeki kayıtlarda yeni bir bağlantı görüyor olmalısınız.
7. Hasatçıyı durdurmak için `chia stop harvester` komutunu çalıştırabilirsiniz. 

*Uyarı:*

Bir makineden diğerine `config/ssl` klasörünün tamamını kopyalayamazsınız.
Ana makinenizin her bir hasatçıyı farklı bir hasatçı olarak tanıyabilmesi için her birinde farklı bir TLS sertifika setine sahip olmanız gerekir.
Farklı makineler arasında **aynı** sertifikalar paylaşıldığında hasatçıları düzgün çalışmaması da dahil istenmeyen durumlar oluşabilir.ters failing to work properly when the **same** certificates are shared among different machines.

*Güvenlik Meselesi:*

Beta27 sürümünden itibaren SY dosyaları her bir hasatçıya kopyalanıyor, çünkü daemon onun doğru bir şekilde başlamasına ihtiyaç duyuyor.
Bu istenen bir durum değil, mainnet 'ten sonraki bir sürümde sertifikaları dağıtmanın yeni bir yolu uygulanacaktır.
Herkese açık internet üzerinde hasatçılarınızı çalıştırırken lütfen dikkatli olun.

*Not:*

Şu anda (mainnet sürümü), arayüz hasatçı plot'larını göstermez.
Çalışıp çalışmadığını görmenin en iyi yolu ana makineniz Chia'yı tamamen kapatıp, `config.yaml` dosyasından loglama seviyesini 
`INFO` yapmak ve Chia'yı tümüyle yeniden başlatmaktır. Şimdi `~/.chia/mainnet/log/debug.log` dosyasında kayıtları kontrol edebilir
ve aşağıdaki gibi mesajlar aldığınızı görebilirsiniz:

```
[time stamp] farmer farmer_server   : INFO   -> new_signage_point to peer [harvester IP address] [peer id - 64 char hexadecimal]
[time stamp] farmer farmer_server   : INFO   <- new_proof_of_space from peer [peer id - 64 char hexadecimal] [harvester IP address]
```
new_signage_point mesaji çiftçinin hasatçınıza challenge gönderdiğini belirtir.
new_proof_of_space mesaji hasatçınızın challenge için bir kanıt bulduğunu belirtir.
new_signage_point mesajları new_proof_of_space mesajlarından daha çok olacaktır.

Eğer arayüz kullanıyorsanız ve birden fazla hasatçı çalıştırmak istiyorsanız
* Ana bilgisayarda Chia'yı kapatın
* Bilgisayarda IP adresinizi bulun
* Ana makinedeki `c:\users\(your user name)\.chia\mainnet\config\ssl` dizinindeki SY klasörünün kopyasını oluşturun - SY dosyasını kopyalayın;
`ssl/ca` klasörünü bir ağ sürücüsünde veya USB bellekte paylaşabilir, veya diğer hasatçılara ağ üzerinden gönderebilirsiniz.
`ssl/ca` klasörünü `chia-blockchain` 'in her sürümünde kopyalamanız gerekir -- SY dosyasını hasatçı makineye kopyalayın -- nerede olduğunu bilin.

* Yeni hasatçıda - aşağıdaki adımları izleyin
* Chia yükleyin ve çalışması için normal 24 kelimelik mnemonic anahtarınızı kullanın. Sonra Chia'yı kapatın
* c:\users\(kullanıcı adınız)\.chia\mainnet\config dosyası içerisinde (notepad ile açın9
* enable_upnp: true yazan satırda true yerine false yazın.
* harvester: farmer_peer: host: localhost yazan yeri bulun ve localhost yerine ana bilgisayar IP adresinizi yazın
* 

* Locate the CA folder you copied from main computer-- know its network location.
* Go to command prompt.  Type in or copy **cd C:\Users\(your username)\AppData\Local\Chia-Blockchain\app-1.1.1\resources\app.asar.unpacked\daemon\**
* Make sure the (app-1.1.1) is the current version-- this is when version 1.1.1 is active
* Run `chia init -c [directory]` on your harvester, where `[directory]` is the copy of your **main** machine CA directory and its network location. This command creates a new certificate signed by your **main** machine's CA.
* [directory] this is where you type the link to where your CA folder is stored-- if on the c drive then type for example c:\ca.  The full line would look like `chia init -c c:\ca`
* Then press enter.  Once that process is complete
*Start both your main pc and the new harvester
* The new harvester may take a 10-20 minutes to start the sync procees- it will be a little bit slower- but should start to sync
and will make a full copy of the blockchain to get to normal sync.  You can create plots on that machine or copy plots over.  It will only farm once full sync is completed. 

To know its working
* On your main pc under Farm  tab- at the bottom select "Hide Advanced Options"- scroll down and " Your Harvester Network" wil now show (2) Node ID-- (1) your main pc and (2) your harvester
* also under farm tab under "Last Attempted Proof" your qty of  plots on your harvester will also show up there

**IMPORTANT**
* Chia does upgrades- if you are finding your harvester is not sync with blockchain or wallet-- you may have to re-copy the CA files again from main pc
* Run `chia init -c [directory]` on your harvester, where `[directory]` is the copy of your **main** machine CA directory and its network location. This command creates a new certificate signed by your **main** machine's CA.

If you want to see that it's working in the logs: [Commands Reference - Checking Logs](https://github.com/Chia-Network/chia-blockchain/wiki/CLI-Commands-Reference#checking-logs-and-status)