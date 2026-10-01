PrintNexus
Tek Panel, Tüm Yazıcılar.
PrintNexus, SASA Endüstri 4.0 bünyesinde geliştirilmiş merkezi bir 3D yazıcı filosu izleme ve yönetim sistemidir.
Sistem, 5 farklı markadan 20 adet 3D yazıcıyı tek bir arayüzde bir araya getirir; yazıcı izleme, uzaktan kontrol, canlı kamera yayını, analiz, üretim kayıtları ve maliyet takibi sunar.
[!NOTE] Bu proje, Temmuz–Ağustos 2026 döneminde bir yaz stajı kapsamında geliştirilmiştir. Projenin önceki adı PrintHQ'dur; kaynak kodda ve dosya yapısında bu isme ait bazı eski referanslar hâlâ bulunabilir.

Genel Bakış

PrintNexus, farklı marka ve modellerden oluşan heterojen bir 3D yazıcı filosunu tek merkezden yönetmek amacıyla geliştirilmiştir.
Yazıcıları her üreticinin kendi arayüzü üzerinden ayrı ayrı yönetmek yerine PrintNexus, aşağıdaki işlevler için ortak bir panel sunar:
-Gerçek zamanlı yazıcı durumu takibi
-Sıcaklık ve baskı ilerlemesi takibi
-Uzaktan yazıcı kontrolü
-Canlı kamera yayını
-Üretim ve maliyet takibi
-Analiz ve raporlama
-Rol tabanlı erişim kontrolü
Sistem, modüler bir adaptör mimarisi kullanır; bu sayede farklı yazıcı markaları ve iletişim protokolleri ortak bir backend yapısına entegre edilebilir.

Özellikler
-5 markadan 20 yazıcının tek panelden izlenmesi
-WebSocket ile gerçek zamanlı güncelleme; bağlantı koptuğunda arayüz otomatik olarak yeniden bağlanır
-Yazıcı detay penceresi: canlı kamera, sıcaklıklar, baskı ilerlemesi ve yazıcıya özel analiz
-Uzaktan kontrol: desteklenen yazıcılarda baskıyı duraklatma, devam ettirme ve durdurma
-Canlı kamera yayını: yazıcının kamera protokolüne göre HLS veya MJPEG
-Rol tabanlı erişim kontrolü (RBAC): admin, operator ve viewer rolleri
-Analiz ve raporlama: marka ve yazıcı bazında sıcaklık geçmişi, baskı sonuçları ve verimlilik grafikleri; TXT, Excel ve CSV rapor indirme
-Üretim ve maliyet yönetimi: filament ve elektrik tüketimine göre üretim maliyeti, satın alma bedeline göre kâr hesabı; Bambu Lab yazıcılardan son baskı verisinin otomatik çekilmesi
-Excel dışa aktarma ve TL / USD gösterimi
-Şifre sıfırlama: güvenlik kelimesi ve yönetici onayıyla
-Denetim kaydı: kullanıcı ve yazıcı yönetimi gibi kritik işlemlerin kaydı
-Electron masaüstü uygulaması: tek başına ya da merkezi bir sunucuya bağlanan istemci olarak
-Koyu / açık tema

Desteklenen Yazıcı Filosu
Marka	      Model	            Adet	           İletişim	           Kamera
Bambu Lab	  X1 Carbon	        9	               MQTT (TLS)	         HLS
ZAXE	      X4               	4	               WebSocket	         HLS
Ultimaker	  S5	              2	               HTTP / REST	       MJPEG
Raise3D	    Pro3	            4	               HTTP / REST	       —
FlashForge	Guider 2S	        1	               TCP soket	         MJPEG
Toplam		20	

Yazıcıya özgü iletişim, adapters/ dizinindeki özel adaptörler aracılığıyla yürütülür.
Ultimaker yazıcılar salt okunur izleme modunda çalışır. Raise3D adaptörü hazırdır ancak bu yazıcılara henüz bağlanılamamıştır; ayrıntılar Bilinen Sınırlamalar bölümündedir.





Mimari
┌──────────────────────────────────────────┐
│            Fiziksel Yazıcılar            │
│            5 marka · 20 cihaz            │
└─────────┬─────────────────────┬──────────┘
          │ MQTT · WebSocket    │ kamera akışı
          │ HTTP · TCP          │ RTSPS · H.264 · MJPEG
┌─────────▼───────────┐         │
│      adapters/      │         │
│     Bambu · ZAXE    │         │
│ Ultimaker · Raise3D │         │
│        Guider       │         │
└─────────┬───────────┘         │
          │ ortak arayüz        │
┌─────────▼─────────────────────▼──────────┐
│                server.js                 │
│      Express API · WebSocket · RBAC      │
│     Kamera: FFmpeg/HLS · MJPEG proxy     │
│       Üretim, Maliyet ve Raporlama       │
└─────────┬─────────────────────┬──────────┘
          │ better-sqlite3      │ REST + WebSocket
┌─────────▼───────────┐  ┌──────▼──────────────┐
│     database.js     │  │       Frontend      │
│        SQLite       │  │   HTML · CSS · JS   │
│ Telemetri · Olaylar │  │  Chart.js · hls.js  │
│   Üretim Kayıtları  │  │ Tarayıcı · Electron │
└─────────────────────┘  └─────────────────────┘








Sunucu, yazıcı yapılandırmasına göre her yazıcı için uygun adaptörü oluşturur. Her adaptör ortak bir arayüz uygular; böylece ana sunucu, farklı yazıcı markalarıyla onların kendi protokollerine bağımlı olmadan iletişim kurabilir.
Kamera görüntüleri adaptörlerden bağımsız, ayrı bir kanaldan akar: Bambu Lab ve ZAXE yayınları sunucuda FFmpeg ile HLS biçimine dönüştürülür, Ultimaker ve Guider görüntüleri ise MJPEG olarak sunucu üzerinden arayüze aktarılır.

Roller ve Yetkiler
Yetki	                                   admin	operator	viewer
Paneli ve yazıcıları görüntüleme	        ✓      	✓      	✓
Yazıcı kontrolü	                          ✓	      ✓	      —
Raporlar ve ayarlar                      	✓	      ✓     	—
Yazıcı ekleme ve silme	                  ✓	      —	      —
Kullanıcı yönetimi ve şifre sıfırlama     ✓	      —	      —
onayı	

viewer rolünde yazıcıların bağlantı bilgileri gizlenir veya maskelenerek gösterilir.

Teknoloji Yığını
Katman                 	  Teknoloji
Backend                 	Node.js, Express.js
Veritabanı	              SQLite (better-sqlite3)
Kimlik doğrulama	        JSON Web Token (JWT)
Gerçek zamanlı iletişim	  WebSocket (ws)
Yazıcı iletişimi	        MQTT, WebSocket, HTTP/REST, TCP
Kamera yayını           	FFmpeg, HLS, MJPEG
Frontend                 	HTML, CSS, Vanilla JavaScript
Grafikler	                Chart.js
Video oynatıcı	          hls.js
İkonlar	                  Lucide Icons
Dosya dışa aktarma	      XLSX (SheetJS)
Dosya yükleme	            Multer
Masaüstü uygulaması	      Electron, electron-builder

Proje Yapısı
PrintNexus/
├── adapters/                  # Yazıcı markalarına özel adaptörler
│   ├── base.js                # Ortak adaptör arayüzü
│   ├── index.js               # Yazıcı türüne göre adaptör seçimi
│   ├── bambu.js               # Bambu Lab (MQTT)
│   ├── zaxe.js                # ZAXE (WebSocket)
│   ├── ultimaker.js           # Ultimaker (HTTP/REST)
│   ├── raise3d.js             # Raise3D (HTTP/REST)
│   └── guider.js              # FlashForge Guider (TCP)
├── assets/                    # Proje varlıkları
├── electron/                  # Electron masaüstü uygulaması
│   ├── main.cjs               # Pencere ve sunucu başlatma
│   └── preload.cjs
├── public/                    # Frontend dosyaları
│   ├── index.html             # Ana panel
│   ├── app.js                 # Ana panel mantığı
│   ├── login.html             # Giriş ekranı
│   ├── analysis.html          # Marka bazlı analiz ve raporlar
│   ├── analysis.js
│   ├── uretim-kayitlari.html  # Üretim kayıtları ve maliyet
│   ├── about.html             # Proje hakkında
│   └── style.css              # Ortak stil ve tema
├── database.js                # Veri katmanı
├── printers.json              # Yazıcı envanteri
├── server.js                  # Ana sunucu
├── package.json
├── package-lock.json
├── .gitignore
└── PrintNexus.md              # Ayrıntılı teknik dokümantasyon

Başlarken

Depoyu klonlayın ve bağımlılıkları yükleyin:

npm install

Yazıcılara ait bağlantı bilgileri (IP adresleri, şifreler ve erişim kodları) gizli tutulduğu için depoya dahil edilmemiştir. Uygulamayı çalıştırmadan önce bu bilgilerin yerel olarak tanımlanması gerekir.

Çalıştırma 

npm start

Sunucu varsayılan olarak şu adreste çalışır:

http://localhost:3000

İlk çalıştırmada varsayılan kullanıcı hesapları otomatik olarak oluşturulur. Sistemi kullanıma almadan önce bu hesapların şifrelerini değiştirin.

Electron

Proje masaüstü uygulaması olarak da çalıştırılabilir:

npm run dev

Uygulama iki modda çalışabilir:

Birincil mod: Sunucu, uygulamanın içinde başlatılır.
İstemci modu: Uygulama yerel sunucu başlatmaz, ağdaki merkezi PrintNexus sunucusuna bağlanır. Böylece birden fazla bilgisayar aynı sunucuyu kullanabilir.

Windows kurulum paketi oluşturmak için:

npm run build:win

Yapılandırma
Yazıcı envanteri printers.json dosyasında marka gruplarına ayrılmış olarak tutulur. Her yazıcının panelde görünen bir adı (id) ve markasını belirten bir türü (type) vardır; sunucu bu türe göre ilgili adaptörü seçer:

type	       Adaptör
bambu	       adapters/bambu.js
zaxe	       adapters/zaxe.js
ultimaker	   adapters/ultimaker.js
raise3d      adapters/raise3d.js
guider	     adapters/guider.js

IP adresleri, şifreler ve erişim kodları gibi hassas değerler bu dosyaya yazılmaz ve depoda yer almaz.
Yönetici yetkisine sahip kullanıcılar yeni yazıcıları arayüzdeki Yazıcı Ekle penceresinden de ekleyebilir.

Bilinen Sınırlamalar
-Raise3D Pro3 yazıcılara, cihazlardaki secure password ayarı nedeniyle henüz bağlanılamamaktadır.
-Ultimaker yazıcılar salt okunur olarak izlenir; uzaktan komut gönderilemez.
-PrintNexus yazıcılara yeni baskı işi göndermez; Başlat düğmesi duraklatılmış bir baskıyı sürdürür.
-Bambu Lab yazıcılar yerel ağ modunda kullanılan filament miktarını bildirmediği için üretim kayıtlarında gramaj elle girilir.
-Arayüz kütüphaneleri CDN üzerinden yüklendiğinden, arayüzü açan bilgisayarın internet erişimi olmalıdır.

Dokümantasyon
Sistemle ilgili ayrıntılı teknik bilgiler proje dokümantasyonunda yer alır:
PrintNexus.md























