# LCWProject
Bu projenin amacı iki basit HTML web sayfasının nginx image'ı ile ayağa kaldırılması ve bu ayağa kaldırılan servislerin trafiğinin load balancer ile trafiğinin kontrol edilmesi üzerinedir.

Öncelikle VS Code kullanılarak, web1 ve web2 adında iki klasör oluşturulmuş ve sonrasında index.html dosyasının içine basit bir HTML kurulmuştur.
web1 içerisindeki HTML kodunda "sunucu1", web2 içerisindeki HTML kodunda "sunucu2" yer alacak şekilde doldurulmuştur.

<img width="733" alt="web1html" src="https://user-images.githubusercontent.com/103995604/164645463-99fce572-81b0-40b6-b7cc-f142ae6809b3.PNG">
<img width="732" alt="web2" src="https://user-images.githubusercontent.com/103995604/164645477-a750116f-c3ed-4ba1-9563-e76e54f6e139.PNG">

Kurulan bu HTML dosyaları için web1 ve web2 dosyaları içerisinde ayrı ayrı olacak şekilde Dockerfile oluşturulup aşağıdaki kod yazılmıştır.

<img width="730" alt="webDockerfile" src="https://user-images.githubusercontent.com/103995604/164646316-37449c4a-6428-4b46-adea-a50e35749131.PNG">

Kodun ilk satırındaki "FROM nginx" ibaresi nginx image'sinin kullanılacağını, üçüncü satırındaki "COPY . /usr/share/nginx/html" ibaresi ise nginx image'ı içerisindeki default HTML klasörü olan: "/usr/share/nginx/html" üzerine sayfa içerisinde bulunan (web1 ve web2 için farklı) index.html dosyasını yazdırır. ("." ibaresi bu sayfa içerisindeki dosyalara bak anlamına geliyor.) 

Bu sayfanın nginx ile çalışıp çalışmadığını kontrol etmek için .yml uzantılı bir docker-compose dosyası çalıştırıp aşağıdaki gibi görülebilir:

<img width="731" alt="web1" src="https://user-images.githubusercontent.com/103995604/164647787-50bf3460-79af-41ac-84f3-a18a5811a8f4.PNG">

Console'da bu şekilde çalıştırmak için "docker-compose up -d" yazılabilir. Yazıp entera bastığımızda herhangi bir tarayıcıya gidip localhost:5000 yazıldığında "sunucu1" yazan local bir sayfayla karşılaşılacaktır.

Buradaki build işlemi ./web1 'de docker compose'un yaptığı görev "." yani bu dosya içerisinde "web1" adlı dosyayı aç ve Dockerfile'ı kur. Daha sonrasındaki port işlemi ise linux makinadaki 80 portunu (nginx çalışan port) bu makinadaki 5000 portunda (localhost:5000) çalıştır.

Yukarıda kurulan 2 local web sayfasının trafiğini izlemek için load balancer kurulması gerekir. Bu nedenle proje sayfasının içerisine loadbalancer adlı bir dosya oluşturuldu. Sonrasında nginx.conf uzantılı bir dosya oluşturup burada hangi web sayfalarının trafiğini izlemek istendiğimizi belirtmeliyiz.

<img width="732" alt="loadbalancer" src="https://user-images.githubusercontent.com/103995604/164649505-07da8803-ffa5-42c9-9627-b8211c57893a.PNG">

1-4 satırları arasında görülen kod trafiğini izlemek istediğimiz web sayfalarını göstermektedir.

Daha sonrasında ise loadbalancer dosyası içerisinde bir Dockerfile oluşturmamız gerekmektedir. Bunu oluşturuyoruz ve içerisine:

<img width="733" alt="DockerfileLOADB" src="https://user-images.githubusercontent.com/103995604/164650886-4511b7d2-0297-47dd-b59b-bf55e5d5a3e3.PNG">

yazıyoruz. Burada görüldüğü üzere, nginx image'ı kullanılmıştır ve default .conf dosyası değiştirilerek sistemimize uygun hale getirilmiştir.

Tüm bu ayarlamalardan sonra docker-compose.yml dosyasının hazırlanıp sistemin ayağa kaldırılması işlemine gelinecektir.

<img width="731" alt="Ekran Alıntısı" src="https://user-images.githubusercontent.com/103995604/164651402-614c008e-6f5a-440f-aedb-398564f53090.PNG">

.yml uzantılı sistemi kaldıracak dosyanın içerisinde web1, web2 ve loadbalancer farklı portlarda çalışacak şekilde kurulmuştur. Sistemin çalışmasını görmek için her servis açılıp görülebilir. console ekranında:

<img width="705" alt="Ekran Alıntısı" src="https://user-images.githubusercontent.com/103995604/164652261-497149c6-4430-4336-9e8d-d57ec8dab959.PNG">

<img width="714" alt="Ekran Alıntısı2" src="https://user-images.githubusercontent.com/103995604/164652284-e849ce4b-3e63-4284-ad46-a351f05af9b4.PNG">

yazıldığında, container çalıştırılmış oldu. Docker desktop uygulamasında: 

<img width="749" alt="dockerdesk" src="https://user-images.githubusercontent.com/103995604/164652468-1a396c3d-f439-41ae-8654-5c98fca31fb6.PNG">

görülür. Sistem ayağa kaldırılmıştır. Şimdi ise loadbalancer kurduğumuz porta (http://localhost:7000/) giderek denemeyi yapalım.

<img width="960" alt="11Ekran Alıntısı" src="https://user-images.githubusercontent.com/103995604/164654606-ab60acae-109b-4dbe-94ae-6c4ee9ea4749.PNG">

Porta gidildiğinde ekrana "sunucu1" ibaresi geliyor. loadbalancer/Dockerfile içerisine öncelik olarak web1 yazıldığı için bu gelmek zorundadır. 

<img width="960" alt="22Ekran Alıntısı" src="https://user-images.githubusercontent.com/103995604/164657197-c986db66-1a7d-43a8-9ada-1af23ab6bb0e.PNG">

Sayfayı yenilediğimizde ise "sunucu2" ibaresi geliyor.

Sistemi durdurmak için console ekranına "docker-compose down" girerek containerı durdurabiliriz. 
