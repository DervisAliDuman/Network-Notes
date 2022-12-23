# Network-Notes

## VLAN (Virtual Local Area Network) Nedir?
Sanal yerel ağlar oluşturarak kullanıcıların ve kaynakların gruplandırılmasını sağlar ve bu gruplandırmalar portlara atanır.
Gruplandırma işini donanım ile değil de yazılım işi ile layer 2 (Data link layer) üzerinden hallettiği için ekstra donanım kullanma maaliyeti olmamaktadır.
VOIP cihazlar trunk konfigürasyonuna ihtiyaç duyarlar. (İlerde işinize yarayacak)
###### VLAN paketi 802.1q için şu şekildedir:
6 Byte 
Dest. MAC
(Hedef MAC)
6 Byte
Source MAC
(Kaynak MAC)
4 Byte
802.1q
2 Byte
Ether. Type
46-1500 Bytes
Data
4 Byte
FCS

    • Paketimiz hedef cihazın ve çıktığı cihazın MAC adresini taşır. Hedef MAC bilinmiyor ise ARP sorgusu yollanır ve en başta bu alan FFFF lerden oluşur.
    • 802.1q VLAN taginin içerildiği yerdir.

802.1q içeriği şöyledir:
2 Byte
TPID
3 bit
Priority (CoS)
Öncelik
1 bit
CFI
12 bit
VID
(VLAN ID)

    • TPID, Ethernet type yerine cihaz bu alanı okumaktadır çünkü aynı yere denk gelmektedirler. Bu alanı okuyan cihaz paketin tag’li olup olmadığını anlayabilir.
    • VID, VLAN ID nin depolandığı yerdir ve 2^12 (4096) -2 yani  4094 adet VLAN tag’i alabilmektedir.
    • Priority, o paketin önceliğini belirtir.













## Access port mode nasıl çalışır?
    • Sadece 1 tane VLAN değerine sahip olabilir. (access 100 gibi)
Girişte ->
    1)  Tag’siz paket geldiği durumda porta tanımlı olan access vlan tagini ekler
    2) Tag’li paket Kabul edebilmesi için discard tagged modunun açık olmaması gerekmektedir, bu durumda tag uyuşuyor ise sökülmeden devam edebilir.
Çıkışta -> 
    1) Gelen tag’li paketin tagi access tagi ile uyumlu ise (trunk’dan gelmiş olsa bile) paketteki tag’i söker. 
    2) Tag’siz paket herhangi bir durumda çıkışa gelemez çünkü switchin içinde hiçbir zaman tag’siz bir paket bulunmamakta. 
CASE 1:

Şekilde göründüğü üzere tüm portlar access 10 modunda beklemekteler. 
PC 1 den PC 2 ye veri gönderimi sırasında port 1 ve 3 switchlerin verilerin giriş kısımları, port 2 ve 4 ise verilerin çıkış kısımları olarak düşünülebilir.
PC 2 den PC 1 e veri gönderimi sırasında port 2 ve 4 switchlerin verilerin giriş kısımları, port 1 ve 3 ise verilerin çıkış kısımları olarak düşünülebilir.
Bu durumda PC1 den PC2 ye tag’siz bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket içeride VLAN ID 10 olucak şekilde tag e sahip olur.
    2. Port 2 ye gelen paket VLAN ID si kendi ID si ile uyumlu olduğu için paketi alır ve VLAN ID yi geri çıkarır, paketimiz port 3 e kadar tag’siz bir şekilde yoluna devam eder.
    3. Aynı şekilde tag’siz bir şekilde port3 e ulaşan paket kendi tag’ini (VID 10) ekler sıkıntısız bir şekilde port 4 de bu tag’I çıkartıp paketi PC2 ye iletir.
CASE 2:

Bu durumda PC1 den PC2 ye tag’siz bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket VLAN ID 20 olucak şekilde tag e sahip olur.
    2. Port 2 ye gelen paket VLAN ID si kendi ID si ile uyumlu olduğu için paketi alır ve VLAN ID yi geri çıkarır, paketimiz port 3 e kadar tag’siz bir şekilde yoluna devam eder.
    3. Aynı şekilde tag’siz bir şekilde port3 e ulaşan paket kendi tag’ini (VID 10) ekler sıkıntısız bir şekilde port 4 de bu tag’I çıkartıp paketi PC2 ye iletir.

## VLAN trunk port mode nasıl çal## ışır ?
    • Trunk port Access portunun aksine birden fazla tag bekleyebilir fakat tagsiz paketleri kabul etmez.
Girişte ->
    1)  Tag’li paket gelince trunk için Kabul edilebilir taglerden biri ile uyumlu olması durumunda paketi iletmeye devam eder.
    2) Tag’siz paket Kabul edemez (native vlan aktif değilken).
Çıkışta -> 
    1) Gelen tag’li paketin tagi trunk VLAN değerlerinden biri ile bile aynı ise tagi sökmeden aynen iletmeye devam eder.
    2)  Tag’siz paket gelemez.

CASE :

Şekilde göründüğü üzere port 2 ve 3 trunk modunda 3 VLAN değeri (10/20/30)  beklemekteler. 
Bu durumda PC1 den PC2 ye tag’siz bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket içeride VLAN ID 10 olucak şekilde tag e sahip olur.
    2. Port 2 ye gelen paket VLAN ID’si kendi ID’lerinden biri ile (VLAN10) uyumlu olduğu için paketi alır ve access portunun aksine VLAN ID yi çıkartmadan iletir, paketimiz port 3 e kadar 10 tagine sahip bir şekilde yoluna devam eder.
    3. VLAN 10 tagine sahip olan paket port3 e ulaşır ve tagi sökülmeden port4 de iletilir. Port 4 de  access portu olduğu için bu tag’i çıkartıp paketi PC2 ye iletir.
Diğer durumlar:
    1. Tag’ler trunk portu ile uyuşmaz ise o zaman o paketin o porttan iletimi mümkün olmaz.
    2. Tag’siz paket trunk portuna gelirse paket iletilmez.










## Native VLAN nasıl çalışır?
    • Native Vlan modu açık ise eğer başta hiçbir değişiklik yapmazsanız default VLAN ID değerini Native VLAN ID olarak olarak kullanacaktır.
Girişte -> 
    1) Tagsiz bir paket switchin native modu açık bir trunk portuna geldiyse ve port discard untagged modunda değil ise Switch üzerinde bulunan native VLAN adı verilen VLAN tagini o pakete ekler.
    2) Tag’li paket geldiğinde trunk tagleri ile uyumluluğuna bakılır(native tag ile uyuşmasına bakılmaz) , uymuyor ise paketi geçirmez.
Çıkışta -> 
    1) Gelen tag’li paketin VLAN ID si ile kendi native ID si aynı olduğu durumlarda paket üzerindeki tag’i söker.
    2) Tag’siz paket herhangi bir durumda çıkışa gelemez çünkü switchin içinde hiçbir zaman tag’siz bir paket bulunmamakta. 
CASE 1:

Bu durumda PC1 den PC2 ye tag’siz bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket VLAN ID 50 olucak şekilde native tag e sahip olur.
    2. Port 2 ye gelen paket VLAN ID si kendi Native VLAN ID si ile aynı olduğu için paketi alır ve VLAN ID yi geri çıkarır, paketimiz port 3 e kadar tag’siz bir şekilde yoluna devam eder.
    3. Aynı şekilde tag’siz bir şekilde port3 e ulaşan paket kendi tag’ini (VID 150) ekler sıkıntısız bir şekilde port 4 de bu 150 Native tag’ini çıkartıp paketi PC2 ye iletir.
CASE 2:

Bu durumda PC1 den PC2 ye VLAN 20 tag’ine sahip bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket VLAN ID’sini sadece ve sadece Trunk değerler (10/20/30) için aynısı var mı diye bakar, var olduğunu görüp tag’i sökmeden paketi iletmeye devam eder. (50 tag’inde bir paket gelseydi paketin iletimi sağlanamayacaktı.)
    2. Port 2 ye gelen paket VLAN ID si kendi trunk ID si ile uyumlu olduğu için paketi alır ve paketi port3 e iletir.
    3. Port3 ve 4 ile de kendi tagi uyumlu olduğu için paket başlangıç ID si olan 20 tag’i ile beraber PC 2 ye iletilmiş olur.
! Burda olan sıkıntı şu ki PC ler tag’li paket okuyamaz.  Evet paket PC2 ye geldi ama paketi okuyamayacağı için iletişim sağlanamaz. Ne durumda tag’i çıkartabiliriz ? Port 4 access 20 olabilir di ve bu tag normal olarak trunk da yer almazdı ya da native tag’i 20 olabilirdi.

## Hybrid VLAN mode nasıl çalışır ?
3 tane port değeri kullanılmakta: 
    1. Trunk için beklenen değerler -> Normal bir trunk portunun işlevini yerine getirebilir.
    2. Untagged için beklenen değerler -> Access portu gibi çalışır fakat farklı olarak birden fazla ID bekleyebilir. Çıkış yaparken untagged tag’lerinden biri ile uyumlu bir tag geldi ise o tag’i söküp ilerletir.
    3. Default (Native) için -> Paket girişte tag’siz bir biçimde gelirse Native tag’ini alır ve normalde de olduğu gibi bu tag’den sadece bir tane vardır.
Girişte -> 
    1) Tag’siz paket geldiyse Native(default) VLAN tagini ekler.
    2) Tag’li paket geldiyse tag trunk ya da untagged tag değerlerinden birini sağlıyor ise paketi aynı tag’i ile Kabul eder.
Çıkışta -> 
    1) Tag’li paket geldiyse eğer tag’imiz trunk tag’lerinden birini sağlıyor ise tag’i sökmeden iletir.
    2) Tag’li paket geldiyse eğer tag’imiz access(untagged) tag’lerinden birini sağlıyor ise tag’i söküp paketi iletir.

CASE 1:

Port 1 (Access):  ->  Access 30
Port 2 (Hybrid):  -> Trunk 10/20    -> Untagged 30/40    ->  Default 50 
Port 3 (Hybrid):  -> Trunk 10/20    -> Untagged 30/40    ->  Default 50 
Port 4 (Access):  -> Access 50 
Bu durumda PC1 den PC2 ye tag’siz bir paket göndermek istersek sırasıyla:
    1. Port 1 e gelen paket VLAN ID 30 olucak şekilde Access den tag alır.
    2. Port 2 ye gelen paket VLAN ID si kendi Untagged VLAN ID lerinden biri ile aynı olduğu için paketi alır ve VLAN tag’ini geri çıkarır, paketimiz port 3 e kadar tag’siz bir şekilde yoluna devam eder.
    3. Aynı şekilde tag’siz bir şekilde port3 e ulaşan paket kendi default tag’ini (VID 50) ekler.
    4. Port 4 de gelen tag’li paket in tagi 50 ve port 4’ün Access değeri 50 olduğu için tag’i söker ve iletir. 







## VTP (VLAN Trunking Protocol) ve GVRP ( Generic VLAN Registration Protocol)
    • İkisi de aynı işi yapmakta fakat VTP cisco cihazlar için üretildiğinden sadece cisco cihazlarda çalışabilmekte. GVRP daha generic olduğu için cisco dahil tüm cihazlarda çalışabilmektedir.
    • Birden fazla switchin yer aldığı durumlarda yapılan ayarın diğer switchlere de uygulanması çok zaman alır ve bazı switchleri değiştirmeyi unutabiliriz ya da hatalı değişebiliriz fakat GVRP sayesinde bir switch üzerinde yapılan değişiklik diğer switchlerin de güncellenmesini sağlamakta.
    • Bir değişiklik yapıldığı zaman VTP configuration revision number denen sayaç 1 arttırılır ve diğer switchler de kendi numarasının yetersiz olduğunu görüp o cihazdan gerekli konfigürasyonları alıp kendini günceller.
STP (Spanning Tree Protocol)
    • 802.1 standartıdır ve tüm köprü görevi gören cihazlar arasında tek aktif bir bağlantı kalması için bazı portları bloklar. Bu sayede öngörülemez loopların önüne geçilir.
    • STP kullanılan ağlarda her bir ağ başına bir tane root bridge, her bir non-root bridge’de bir tane root port ve her bir parçada trafiğin geçmesi için bir tane designated port (forwarding state) bulunur. (Root bridge’nin tüm portları designated port modundadır)
    • Bridge ID aslında köprü görevi gören cihazın MAC adresidir.
BPDU (Bridge protocol data unit): Bu paketler STP ile ilgili bilgileri tutar. Mesela BID tutar. BID ise bridge’e ait olan priority ve MAC adres değerlerinin bulunduğu bir tag’dir. (Priority 16 bit , MAC 48 bit olacak şekildedir.) İlk kısım (significant) priority bit den oluştuğu için priority, MAC adresten önemlidir.
Nasıl çalışır? 
    1. Başlangıç olarak tüm switchler kendilerinin root bridge olduğunu iddia eder ve hepsi BPDU yollar.
    2. Her 2 saniyede bir BDPU sinyalleri yollanır ve bu sayede en düşük BID ye sahip bridge, root bridge olarak seçilir, diğerleri kendilerinin root olmadığını anladıklarından itibaren root bridge’in (O anlık root bridge değil de kendisinden daha iyi bir BID değerine sahip bir bridge varsa onu iletir) verisini iletirler.
    3. Root Bridge portlarını designated port moduna alır ve forwarding (iletim) state’ine geçiş yapar. Sonra diğer bridglere hello sinyali yollar.
    4. Her hello sinyali alan bridge(switch) önceki maaliyeti kendi maaliyetini de ekleyip iletir. Hello time içerisinde ulaşılamayan bir bridge olduysa STP topolojisini değiştirmek için sinyal yollar.
    5. Tüm bridge’ler aslında root a ulaşımı açısından maaliyeti en düşük olan yolu tercih edecek şekilde hesaplar.
    6. En düşük maliyetli yolun hesaplanmasıyla oluşan bilgiler doğrultusunda portlara roller atanmaya başlanır.
    7. Switchlerin root bridge olarak seçilen switch ile iletişim cost’u en düşük olan bir port root port olarak seçilir ve her switchin kendine özel bir root portu olur.
    8. Designated portlar veri iletebilen portlardır ve root bridge üzerindeki tüm portlar designated porttur. Veri iletebilme durumu forwarding state olarak bilinir. Diğer switchler için root bridge ile bağlantısı olmayan her trunk bağlantının bir ucundaki port designated port olarak seçilir. Önemli olan detay root port olarak seçilen portları designated port seçemiyor oluşumuzdur.
    9. 2 Designated port çakışınca en yüksek cost’a sahip olan blocklanır, eğer costlar eşit ise en yüksek BID ye sahip olan port blocklanır. Eğer aynı BID ye sahipse (kendisi ile loop var ise) o zaman port numarasına bakılır.
    10. RSTP için root bridge haricindeki diğer switchlerin root ya da designated olarak seçilmeyen portları alternate port olarak seçilir ve blocking state de olduğu için gelen paketleri yok sayar.
Convergence Time: STP protokolünün basamaklarının tümünün (Root bridge seçimi, port modlarının atanması…) tamamlanması ya da sproblem oluşması durumunda yeniden STP düzeninin oluşması için geçen zamana denir.
STP Timers : 
    • Hello timer (default 2 second): Sistemin hala ayakta olup olmadığını kontrol etmeye yarar. Hello paketleri içerisinde BDPU paketi bulunur. Sürekli olarak yollanır.
    • MaxAge (10xHello time): Switch’in hello sinyali bekleme süresidir. Bu süre içerisinde hello mesajı gelmesi beklenir. Gelmemesi durumunda Topology change sinyali yollanır.
    • Forward delay (default 15 second): Switch’in portlarının state değişimi arasındaki geçen zamandır.
STP Convergence In General: 
    • STP protokolünün basamaklarının tümünün (Root bridge seçimi, port modlarının atanması…) tamamlanması için geçen zamana denir.
    • Eğer BDPU aging kullanılıyor ise ve bir link hatası algılandıysa (maxage içerisinde BDPU sinyali alınamadıysa) o zaman tekrardan yeni bir STP düzeni kurulacağı için convergence basamaklarının bazıları yapılır. Bunun için gerekli süre ise en az 2xForward_Time, en fazla ise 2xForward_Time + Max_age kadardır.
    • Bir değişiklik sonucu topology change sinyali geldi diyelim. Bu durumda en ufacık değişiklik bile bir sürü alternatif yol ortaya çıkarmış olabilir. Bu yüzden yeni costlar yeni yollar öğrenilebilmesi adına Mac adres table dahil tüm table’lar temizlenip baştan doldurulması gerekmektedir.
STP Topology Change Sinyali gelince ne olur?
1->> Sıkıntı çıkan Bridge sinyali root portu aracılığı ile diğer bridge’lere iletir taki root porta bu sinyal ulaşana kadar iletilir. (Upstream)
2->> Bu TC BDPU sinyali root a ulaştıktan sonra, root’dan TCN Acknowledge bit’lerin ayarlanması için sinyal yollanır.
3->> Root dan TCN(Topology Change Notification) sinyali diğer bridglere yollanır (downstream). Bu flag’in set olma süresi Max_Age + Forward_Time kadardır.
4->> Mac adreslerinin hepsi temizlendiği için yeniden Mac adresleri öğrenilmeye başlanır ve bu süre Forward_Time kadardır.

2xForward_Time < toplam convergence time < 2xForward_Time + Max_age.

## STP PortFast
Loop olma riski olmayan son kullanıcılar (Host ya da enduser denebilir) BDPU paketi kullanmazlar. Portumuzun sırayla blockingden diğer port state’lerine geçiş yapmalarına gerek olmadığı için STP’ye çok fazla zaman ayrılmasını önler. TCN (Toplogy change notification) portFast açık olan porttan yollanamaz ve bu porttan switch bağlanamaz.
STP Extensions: UplinkFast & Flexlinks
UplinkFast alternate port mantığını kullanır. Eğer root port bağlantısında sorun yaşanırsa alternate port, root port gibi davranır. Eğer birden fazla blocked port var ise en düşük cost’a sahip olan root port seçilir. Frame gönderilme sıkılığını {“spanning-tree”,” uplink-fast”,”max- update-rate”} komutları ile ayarlayabiliriz. Link hatasından kurtulmak bu sayede bir saniyenin altında gerçekleştirilebilmektedir.
Bu sayede en az 30 saniye (2Xforward) sürecek convergence time süreyi kullanmaya gerek kalmadı.
(Bazı sıkıntıları varmış anlamadım)
Flexlink’i uplinkFast in STP siz bir versiyonu gibi düşünebiliriz. 2 port arasında link oluşturmak için “switchport backup” komutu kullanılır ve linklerdeki STP devre dışı bırakılır. Standby link hala gelen paketleri yoksayar ve mac address table I doldurmazken diğeri depolar. Birincil link eğer fail olursa standby link aktifleşir ve mac adressleri taşır. Bu işlemler için gerekli komutlar -> “mac address-table move {receive|transmit}” ve “switchport backup interface x/y mmu” dur.
Flexlink STP yi deaktif edip root port değişiminde topology change prosedüründe bize yardımcı olur fakat bazı problemleri vardır. (Bilmiyorum bana da öğretin)
Link fail case without STP extension
Diagram üzerinden açıklamak istersek. 
Bridge A -> root bridge
B’nin bir portu blocked durumda.
Bridge C -> Inferior BDPU ile C nin root olduğunu iddia ediyor. (Inferior BDPU denme nedeni önceki yoldan daha maliyetli bir yol seçmiş olmasından kaynaklanıyor ve zaten diğer switchler bu paketin inferior olduğunu kendinde bulunan önceki root BID ile karşılaştırarak anlayabilmektedir.)
    1- C sürekli olarak B ye kendisinin root olduğuna dair BPDU yolluyor fakat B deki blocked port bunları yok sayıyor çünkü zaten diğer taraftan daha düşük olan root BID’si gelmekte. Bu nedenle aslında ordan BDPU almamış gibi oluyor ve Max_Age süresince bekliyor.
    2- Max_Age süresi bittikten sonra artık C deki port listening state’ine geçiyor ve A dan gelen BDPU yu değerlendirmeye başlıyor. Sırasıyla 3 state e giriyo; listening ile A nın root olduğunu anlıyor, learning ile MAC adreslerini baştan dolduruyor sonra da forwarding state’e geçiyor.
    3- Bu şekilde en başta max_age bekledik, şimdi ise port state lerinin değişmelerini bekledik. Bu yüzden harcadığımız zaman = max age timer + (forward delay timer) x 2 oluyor.
Superrior BDPU -> Designated modda ve Forwarding state den çıkmış olup blocked porta giden BDPU paketine denir.
STP Extensions: BackboneFast
Diagram üzerinden açıklamak istersek. 
A -> root bridge
B’nin portu alternate port.
C -> Inferior BDPU ile C nin root olduğunu iddia ediyor. 
Sonra B ise önceki root ile olan bağlantısının hala var olup olmadığını kontrol ediyor.
    1- Bridge root portunu ve diğer tüm alternate portlarını seçer. Sonra özel bir mesaj olan Root Link Query (RLQ) yu seçilmiş portlara yollar. BDPU şunları içerir: local BID ve yeni root bridge için sorgulanan Bridge ID.
    2- RLQ yu alan tüm Bridge’ler, Root BID yi control eder ve duruma göre aşağıdakileri yapar.

    a) Eğer Root BID ile şuanki root bilgisi eşleşiyorsa RLQ yu upstream e yollar.
    b) Eğer bu RLQ yu alan Bridge root bridge ise ve root bilgisi eşleşiyor ise tüm designated portlarından pozitif RLQ sinyali yollar.
    c) Eğer gelen RLQ sinyali ile root bridge bilgisi eşleşmiyor ise tüm downstream portlardan negative RLQ sinyali yollar.
    3- En baştaki sinyallerin çıktığı bridge negatif sinyal alır ise portunda depolanan bilgileri geçersiz sayıp listening state e geçiş yapar. Eğer pozitif sinyal alır ise portta depolanmış olan bilgi valid olarak değerlendirilir.
    4- Eğer tüm response’lar negative ise, sorguyu yapan bridge eski root ile bağlantı kaybı yaşadığını belirtir. Böylece local bridge kendisini yeni root bridge olarak ilan eder ve önceden bloklanmış olan portlardan veri almaya başlar. Eski bridge informationlar silinir. Artık Max_Age kadar beklenmesine gerek kalmamıştır.
    5- Eğer en azından bir tane bile RLQ cevabı pozitif ise sorgu yapan bridge en azından root a ulaşabileceği 1 yol olduğunu öğrenmiş olur. Bloklamış olduğu portu listening state’e alır ve ordan STP yi yeniden kurmaya gerek kalmadan haberleşmiş olur.
ÖZET: Normal STP den en önemli ilk farkı bu 5. Madde aslında. Amacımız bu tip durumlarda STP için boşa zaman harcamamak. Burada hem max_age zamanından hem de forwarding_time timerlarından kar ettik. Yani aslında normal STP de 50 saniye sürcek işlem için çok zaman kaybetmedik.
Diğer bir fark ise 4. Maddede ki gibi sadece 20 saniye olan max_age den kar etmemizi sağlıyor. 50 saniye yerine 30 saniye sürüyor çünkü 5. Maddede olduğu gibi max_age beklemek yerine direk stp yi yeniden kurmaya başladık. Bu convergence time sadece 2 x forwarding time kadar sürmektedir.

## Protecting STP 
STP 2 nedenle ataklara hassas:
    1) Root Bridge en düşük BID ye sahip olan seçilmekte (Her zaman optimal ve stabil değil. Kötü niyetli bir şekilde ya da yanlışlıkla sisteme daha düşük BID ye sahip cihaz takılabilir.)
    2) STP, komşu switchlerden gelen BDPU’ları Kabul ederek topolojiyi oluşturur.
Protection için 3 tane mekanizma kullanılır: Root Guard, BPDU Guard, BPDU Filtering
Bunlar Cisco’da default olarak disabled haldedir.
• Root Guard 
Yetkisi olmayan cihazın kendini root olarak tanıtmasını engeller. Eğer root guard açık bir porta daha iyi bir BDPU (superior BDPU) gelirse blocklanır sonra root-inconsistent (tutarsız) hale gelir ve artık frame yollamayı bırakır fakat BDPU’ları dinlemeye devam eder. Superior BDPU gelmeyi bırakır bırakmaz, port eski stp düzeninde devam eder.
Root guard portlara özgüdür ve her port için “spanning-tree guard root” komutuyla ayrı ayrı açılabilir.
Root-inconsistent haldeki portları görüntülemek için” show spanning-tree inconsistentports” komutu kullanılabilir. 
• BPDU Guard 
Bu portfast de hani sadece host bağlanmasını istiyorduk yani loop oluşması olası cihazlar (switch etc.) istemiyorduk. Neden? çünkü portfast STP’yi devredışı bırakmıyor sadece state’lere girmesini ve topology change ile uğraşmasını engelliyor. Hala BDPU’ları Kabul ettiği için loop oluşcak olursa bir anda broadcast storm oluşabilir.
BDPU guard ile ne olursa olsun porta herhangi bir BDPU geldiği an o portu devre dışı bırakır.
• BPDU Filtering
BDPU’ların porttan gönderilmesini engeller. PortFast ile birlikte etkinleştirilip kullanılmalıdır. Konfigürasyonuna göre BDPU geldiği an BDPU filtering 2 şekilde davranabilir:
    1) Eğer filtreleme global olarak etkinleştirilmişse, gelen BDPU portFast’I devre dışı bırakır. Sonra normal STP çalışır mı öyle bir şeyler yazıyor anlarsanız bana da anlatın. Protokoldeyim Derviş adım. Burda yoksam bile numaramı bulun arayın söyleyin bu tip eksikleri. Teşekkür ediyorum çok incesiniz :D
    2) Tüm interface’lerde filtreleme açıldıysa o zaman gelen BDPU droplanır.
Filtreleme açılırken BDPU yok sayılacağından STP esasen deredışı bırakıldığı için dikkatli olunmalıdır. Bağlantı noktası ne hata-devre dışı bırakılacak ne de STP işlemi boyunca ilerleyecektir ve bu nedenle bağlantı noktası döngülere açıktır.


## RSTP (Rapid Spanning Tree Protocol)
    • STP den farklı olarak;
    1) Hello sinyali sadece root bridge’den değil, tüm switch’lerden gelir.
    2) STP’de maxage timer’ı (default 20 sn) vardı ve eğer bu süre içerisinde hiçbir BDPU paketi ulaşmaz ise bir şeylerin sıkıntılı olduğuna karar veriyordu.  Bu olay RSTP için ise 3xHello time kadar beklenir eğer o süre içerisinde bir BDPU paketi gelmez ise o zaman uyarı sinyali yollanır.
    3) RSTP de port forwarding state e geçmeli mi diye karar verirken timer kullanmaz, böylece oradaki zamandan tasarruf edilmiş olur.
Root Port – STP deki gibi Non-root bridge’den root bridge’e giden, cost’u en düşük porttur. Root Bridge’e forwarding yapar.
RSTP BPDU Yapısı:

RSTP’de port rolleri eklendiği için, BPDU yapısına bu roller de eklenmiştir. Flag bölümündeki Port Role kısmında şu seçenekler vardır: Unknown, Alternate / Backup port, Root port, Designated port.


Proposal/Agreement (P/A) mekanizması, designated bir portun çok hızlı bir şekilde forward state’e geçmesini sağlar. Aşağıdaki figurler, bu mekanizmayı göstermektedir.

Proposing: Bir port, discarding veya learning state’deyken bu değer 1 olarak ayarlanır. Proposal field’ı 1 olan mesaj, downstream cihaza yollanır.

Proposed: Komşu cihazın designated portundan BPDU proposal field’ı 1 olan mesajı alan port, designated olan portu forwarding state’e sokar.

Sync: proposed değişkeni 1 olarak ayarlandıktan sonra, proposal’ı alan root port sync değişkenini tüm portlar için 1 yapar. Sync 1 olan non-edge portların hepsi discarding state’e geçer.

Synced: Bir port discarding state’e geçtikten sonra,  eğer bir alternate, backup ya da edge port ise synced değişkenini 1 yapar fakat eğer root port ise diğer portları izler. Diğer tüm portlar synced değişkenlerini 1 yaptıktan sonra, root port da synced değişkenini 1 yapar ve RST BPDU’sunu Agreement field 1 olarak yollar.

Agreed: Designated port, port rolü root port olan ve Agreement field’ı 1 olan RST BPDU’yu aldıktan sonra agreed değişkeni 1 olarak ayarlanır. Bu değişken 1 olur olmaz, port hemen forwarding state’e geçer.
RSTP Port Rolleri
    • RSTP, link çöktüğünde convergence time’ın çok olmaması için bridge’lere farklı port rolleri eklemiştir.
Designated Port - Bağlı olduğu segmentte BDPU paketlerini forward’lar.
Alternate Port – Root Bridge’e yeniden bağlantı kurulabilmesi için bekleyen alternatif porttur. Gönderilen BPDU Paketleri’ni aldıktan sonra, forward’lamak için alternatif bir port olarak bloke olarak bekletir.
Backup Port – Önceden bağlı olan bir bridge için yedek olarak bekletilen port. BPDU paketlerini alır.
Disabled Port – BPDU paketleri almaz.

Port state’leri ise 3 tanedir:
Discarding – port üzerinden data yollanmaz. Port block’ludur. BPDU paketleri alınır.
Learning- Port henüz frame’leri yollamaz. Bu state, MAC adreslerini tabloya ekliyor demektir.
Forwarding- Normal bir şekilde data alımı ve gönderimi gerçekleşir.

## RSTP Sync Process
RSTP caching mekanizması konseptini kullanır. (caching en son kullanılan verileri bir yerde depolayıp sonradan ihtiyaç durumunda tekrardan eski işlemleri yapmak yerine depoladığımız cache memory üzerinden direk istenen veriyi bulup çekmek için kullanılan bir yöntemdir). Eğer bir path fail durumunda ise cache üzerinden alternatif bir path bulunur. Ayrıca RSTP Negotiation (Teklif verme gibi düşün) methodunu kullanır.
Örnek üzerinde daha iyi anlaşılacaktır:

    1- Proposal(teklif) aracılığı ile daha iyi bir root bridge bilgisi geldiği zaman ya da root port değiştiği zaman diğer bridge bunu farkeder. 
    2- Bulunduğumuz bridge yeni root a bağlı olan port hariç tüm designated portları bloklar ve root bridge ile bağlantısı olan port root port seçilir. (Aslında yeni root için downstream yollarını bloklamış olduk).
    3- Root port hariç tüm portlardan downstream yönünde proposal flag içeren BDPU sinyaleri yollanır.
    4- Aynı şeyi downstream bridge’ler de yapar tâki tüm downstream bridge’lerden agreement bit’i ayarlanmış bir BDPU response gelene kadar.
    5- Agreement mesajı root a ulaştığı an upstream deki her bridge downstream portları unblock hale getirir tâki en sondaki bridge e ulaşana kadar. 
! Eğer proposal gelen bridge üzernde daha iyi bir bridge bilgisi var ise proposal mesajını kendi bridge bilgisi ile geri yollar. Burda root için önerilen ama reddedilen bridge için 2 durum var.
    1- Alternate proposal gelen portu bloklar. Bu durumda senkronizasyon işlemi daha fazla devam etmez.
    2- Alernate proposal gelen portu root port seçer. Bu durumda ilk senkronizasyon tam tersi yönde akmaya başlar.
RSTP’de blocked portlar sadece tek bir özel durumda BDPU sinyallerini kabul eder. Bu BDPU sinyalleri başka bir root dan gelmiş ise yukardaki durumlarda olduğu gibi ya yeniden senkronizasyon yapar ya da proposal ı geri yollar. Bunun dışında gelen BDPU’lar yoksayılır.



## RSTP Topology Change
Yandaki figürdeki upstream işlemi TCWhile time (2xHelloTime) kadar zaman alır. Bu sinyal Upstream’den ya da downstream’den gelebilir. Bu durumda aşağıdakiler gerçekleşir.
    1-  TC BDPU gelen portlar haricindeki tüm portlarda MAC adresleri silinir.
    2-  TCwhile timer süresince bunu tekrarlar ve tüm BDPU larda TC bitini set eder.

Burda asıl amacımız mümkün olduğunca hızlı bir şekilde TC işlemini gerçekleştirmek. Mesela root portun bağlantısı kopunca alternate portu root port kullanmamız da ayrı bir optimizasyondur.
Link Down işlemi sonucunda extra bir değişiklik yapılmaz, örneğim MAC tablosu silinmez. Çünkü bağlantı kopması normalde var olan yolların değişmesine neden olmaz.
Eğer downstream üzerinde hiç alternate port yok ise işte o zaman convergence için herhangi bir iyileştirme yapamamaktayız. 
Edgelinks forwarding yapsalar bile TC oluşturmazlar ve MAC adreslerini silmezler. Bu da performans açısından önemli bir yer alır. (edgelink derken end-userlardan bahsediyor, hani direk forwardinge geçiyor ya o mevzu)
Edgeport -> end-user bağlanmasını bekleyen portdur ve direk forwarding state e geçer. Aksi halde devredışı kalır.
Auto-edge port -> porta end-user bağlanırsa direk forwarding moduna geçer, switch falan bağlanırsa o zaman sırası ile port modlarından geçer.

## RSTP Illustrated Ring topology
    A) Failure 1-> Link Failure in Ring Topology:
Çok önemli not !!! -> Eğer A ile B arasındaki link fail olsa idi o zaman hiçbir değişiklik olmayacaktı çünkü zaten blocked bir link söz konusuydu. Ama eğer resimdeki örnekte olduğu gibi başka bir yerde kopma yaşanırsa işler ilginçleşmeye başlıyor ve şu basamaklar meydana geliyor:
1->  Cloud 2 ile root bağlantısını sağlayan bridge 3xHello world sinyali süresince bekleyip sonuç alamayınca diyor ki root ile olan bağlantıda bir sıkıntı oluştu. Sonrasında kendini root ilan ediyor.
2-> Bridge Y’nin önceliğinin daha fazla olduğu bir senaryoda hem Cloud 2 hem de Cloud 3 için root seçildiğini düşünelim. Bundan sonra ise bu bilgiyi downstream aracılığı ile paylaşıyor.
3-> Y bridge’in R dan (gerçek root) uzak olması sistemin yeniden adapte olmasını zorlaştırır.
(Bunları ben kendim varsaymadım koca doküman bunları kendi varsaymış. Her cümlede “lets assume” kalıbı var)
4-> Bridge B, Y den gelen sinyali alır ve A ya daha iyi bir root BDPU su için sorgu yollar. Bu bilgi Root a ulaştığı zaman bridge B tekrardan gerçek root ile senkronize hale gelebilir ve Blocked portu açar.
5-> Blocked port’un açılmasıyla birlikte TC flagi yollanır

    B) Failure 2-> Link Failure in Ring Topology for Root Bridge:
 
Üstteki örnekten (failure 1) farkı şu ki, bu sefer sadece tek taraftan root a bağlantı kopması deği. Bu olayda 2 tarafta root ile bağlantısını kaybediyor.

İki taraf derken biri Y ve B, diğer taraf X ve A. Bunlar üstteki gibi kendi root bridge’lerini seçerler.
Burdan sonra asıl root bridge olmadığı için bu seçilen BDPU’lar arasında en iyi olan root bridge seçilir.
Sağdaki resimde gördüğünüz üzere cache’de duran Bridge R’ın BDPU su hala etrafta dolaşmakta. Bu durum işlerin yavaşlamasına neden olmakta çünkü diğer bridge’ler hala R’ın yaşadığını sanmaktadırlar.
2 ayrı segmentte root seçtiğimiz için ise 2 kat yavaşlama riskimiz var. Bu nasıl oluyor? Y’nin en iyi BID ye sahip olduğu senaryoda 2 segment kendi arasında root seçtikten sonra durum kimin önce kendini tanıttığına kalıyor (race condition). Eğer önce A, Y’nin bilgisini alır ise convergence time iki kat artmış oluyor. Tam tersi durumda normal convergence time kadar bekliyoruz.

## TOPLANIN OLAY VAR (RSTP WORST CASE SENARIO) Counting to Infinity Problem










Üstteki durumda R başlangıç olarak Root ama bağlantısı kopuyor.
BID si en düşük bridge 1 olduğu için onun yeni root olması lazım ama bir de ne olsun. RSTP nin doğası gereği her bridge cache’de tuttuğu root BID sinin bulunduğu BPDU’yu paylaşıyor R is root diye.
Bu BPDU dönüp dolanıp Bridge 1’e geri dönüyor ve bridge 1 de masum bakıyo ve kendi BID’sinden düşük olduğu için R is root diyor. Halbuki R bridge’i aslında piyasada yok sadece BPDU’su dolanıyor. 
Bu problem Max_Age kadar devam etmektedir, sonrasında Bridge 1 durumu anlayıp yeni topolojiyi oluşturması gerekmektedir. O zamana kadar eski root olan Bridge R’ın BPDU’su paylaşılmaya devam edilecektir.

(burayı biraz açmam gerekcek)




## MSTP (Multiple Spanning Tree Protocol)
STP ve RSTP de bloklu portlardan trafik akmamakta ve biz bunu istemiyoruz ve önüne geçmek istiyoruz çünkü trafik ne kadar düzenli dağılırsa o kadar efektif bir kullanım sağlanır. MSTP ile aynı ağ üzerinde birden fazla STP topolojisi (birden fazla tree) oluşturup her bir STP ağı için ayrı portlar bloklu şekilde davranması mümkün hale geliyor. 
MSTP nin en temel farklarından biri, root bridge seçerken artık BID ye bakmıyoruz. Kendimiz MSTP yi ayarlarken her cihaza o ağaç için gerekli önceliği veriyoruz.
Bir diğer fark ise MSTP ile farklı VLAN’lerle farklı ağaçlar kendi içlerinde haberleşebilmekteler.
Soldaki örnekte olduğu gibi her tree için farklı root olması mümkündür.
Aynı şekilde her tree için blocked, root, designated portlar da ayrı ayrı bulunabilmekteler.

Bu örnekte tree 1 için bloklu port switch 2’de Fa 0/1 portu, switch 0’da Fa 0/2 portudur.

## RING Topology
Bus topolojisinin loop şeklindeki versiyonu gibidir.
PTP (Peer to Peer) Topolojisindedir, yani kimsenin kimseye üstünlüğü ya da önceliği yoktur.
Bağlantı şekilleri unidirectional (tek yönlü) ya da bidirectional (çok yönlü) şekilde olabilir.
Token’ler aracılığı ile haberleşme sağlanır.
Unidirectional haberleşmede bağlantı kopması durumunda haberleşme bozulacaktır.
Paketin heryere ulaşması ve her yoldan geçmesi performansı düşürmektedir.
MRP (Metro Ring Protocol) (bekle hemen bakma benim de kafam karışık)
Layer 2 loop’ları engeller ve hızlı bir reconvergence hızına sahiptirler. STP ye alternatiftir ve özellikle MAN(Metropolitan Area Networks) için kullanımı STP ye göre daha iyidir çünkü: 
    • STP de max. 7 node bulunabilirken Metro Ring daha çok node a sahip olabilmekte.
    • STP de convergence time çok daha yavaş. 
MRP’de de STP deki gibi bloklu portlar bulunur. Herhangi bir link kopması durumunda bloklu portlar açılıp yeniden bağlantı kurulabilir. Bloklu portlar forwarding yapamaz. 



## QoS (Quality of Service)  (bekle hemen bakma benim de kafam karışık)
Çok önemli dataların belli bir öncelik ve güvenlik ile yollanmasını sağlar. Qos bazı mekanizmalar sayesinde Network trafiğini kontrol eder. Genellikle IPTV’lerde, oyunlarda, streamingde vb. yerlerde kullanılır.
Network trafiğini kontrol eder dedik, peki nedir bu trafik çeşitleri :
    • Bandwith: Linkin hızıdır. QoS router’a bandwith i nasıl ve ne kadar kullanabilceğini ayarlayabilir.
    • Delay : Gönderilen paketin source ile destination arasında geçen zamana denir. QoS her tip trafik için priority queue ile organize eder.
    • Data Loss : Bağlantı yoğunluğu(izdihamı) nedeni ile oluşan paket kaybıdır.
    • Jitter : Trafik düzensiz zamanlarda veya yanlış sırada geldiğinde meydana gelen parçalanmayı açıklar ve hızını gösterir.
Types of Delays:
    • Serialization Delay: Interface’in verileri fiziksel bir ortama kodlaması için gereken süreyi ifade eder.
Formülü ise->>  ( # of bits ) / (bits per second)  
    • Propagation Delay:  Kablo üzerinde bir bitin uçtan uca gitmesi için gereken süreyi ifade eder.
    • Forwarding (or Processing) Delay: bir router veya switch’in bir paketi bir giriş (ingress yazıyordu) kuyruğu ile bir çıkış (engress yazıyordu) kuyruğu arasında taşıması için gereken süreyi ifade eder. 
    • Queuing Delay: Çıkış kuyruğunda önceden kuyruğa alınmış paketlerin serialization işlemini beklerken geçen süredir.
    • Network (Provider) Delay: WAN sağlayıcısının bulutunda geçen süredir. Bulut’un structure’unu bilmek zor olacağı için bu ağ gecikmesini hesaplamak da çok zor olabilmektedir.
    • Shaping Delay: Tıkanıklık (congestion) oluşması nedeniyle paketler drop olabilmekte ve bu sorunun düzeltilebilmesi için trafiği yavaşlatan mekanizmalar kullanılır. Bu mekanizmalar tarafından olan gecikmelere denir.
QoS Methodologies
    • Best Effort QoS: Aslında bu en basit tabirle QoS olmama durumudur. Trafiği First come First serve mantığı ile yönlendirmekteyiz. Basit implement edilir ve default olarak switchlerde bu yöntem kullanılır. Internet de bu şekilde çalışmaktadır. Anlayacağınız ekstra bir şey yapmıyor.
    • Integrated Services (IntServ) QoS: Aynı zamanda end-to-end ya da hard QoS olarak da bilinir. An Admission Control protocol requeste kaynakları allocate ettikten sonra yanıt verir. Kaynaklar belli bir request için allocate edilmez ise reddedilir. Tüm cihazlar QoS desteklemelidir. 2 yönden kullanması çok da iyi değildir (unscable): Reserve edilebilcek sınırlı bandwith var ve çok fazla ek yükü var. RSVP (bu ne?) bu protokolün bir örneğidir.
    • Differentiated Services (DiffServ) QoS: Trafik tipleri spesifik sınıflara göre organize ediliyor, sonrasında kendi sınıflandırılmalarına göre identify edilir. Bu sınıflandırmaya göre servis edilir. Soft QoS olarak da bilinir çünkü garanti bir şekilde amacımıza ulaştıramamaktadır.
QoS Tools
    • Classification and Marking: Classification is a method of identifying and then organizing traffic based on service requirements. This traffic is then marked or tagged based on its classification, so that the traffic can be differentiated.
    •  Queuing : Bildiğin sıralama mekanizması ama bunun bazı çeşitleri var: • First-In First-Out (FIFO) • Priority Queuing (PQ) • Custom Queuing (CQ) • Weighted Fair Queuing (WFQ) • Class-Based Weighted Fair Queuing (CBWFQ) • Low-Latency Queuing (LLQ)
    •  Queue Congestion Avoidance : Anlamadım


## DHCP (Dynamic Host Control Protocol)  
Otomatik olarak IP atanmasını sağlıyor. Hani sen switch’e ilk bağlanırken bilgisayara IP atıyorsun ya, işte ona gerek kalmıyor, her bilgisayara ya da cihaza tek tek statik IP atman çok zaman alırdı yoksa. Aynı wifi’ye bağlanırken yazan gibi, telefonun aslında istemci; sunucudan DHCP ile otomatik bir IP adresi istiyor ve sen ekranda “IP adresi alınıyor…” yazısı görüyorsun. 
#RULE: Bilgisayar dünyasında en önemli şeylerden biri kullanıcıyı gereksiz detaylara sokmamaktır. Kullanıcının ağa bağlanmak için IP adresi nedir bilmesine gerek yoktur. Bilgiyi kullanıcıdan soyutlama işidir bu aslında ve genelde OOP de karşımıza çıkan abstraction olarak bilinir.
Peki bu işlem nasıl olur? 
    • Eğer DHCP istemcisi olan bir client bağlanırsa, client DHCP server bulabilmek için otomatik olarak DHCP Discover mesajını broadcast olarak yollar.
    • DHCP server bulunur ise DHCP Offer paketi ile cevap verir. Bu paket önerilen IP adresini, subnet maskesini vesaire içerir.
    • Gelen offer client’a ulaşınca alınca DHCP Request (kabul edip etmediği bilgisini taşır) paketi ile cevap verir. Burada amacımız, eğer ağ üzerinde başka server var ise o karışıklığı gidermek için hangi server’ı seçtiğini belirtir. 
    • Son olarak; sunucu, istemcinin (client) sunulan protokol bilgilerini kabul ettiğini bildiren bir DHCPACK ile yanıt verir. Aslında bu aşamada server önerilen IP yi client’a kiralamış olur. (Bu ne alaka dersen, offer paketi gerekli tüm konfigürasyonları içermez. Geri kalan konfigürasyonları DHCPACK içerir.)
Not: DHCPRequest hangi server’ın seçildiği bilgisini içerir. Bunu alan server eğer seçilmemiş ise önceden DHCP Offer ile önerdiği IP adresini kullanılabilir (available) IP havuzuna geri ekler.
Not 2: Client, server’dan kiralamış olduğu IP adresini DHCPRELEASE paketini server’a göndererek, server’ın bu IP adresini available IP havuzuna geri eklemesini sağlar. 





## Random Notlar
    • Switch’de default olarak portlar açık gelir fakat router da bizim açmamız gerekir.
    • IP adreslerinin sonunda 0 olan adresler network ü temsil eder. Router lar aldığı network IP lerini portlarına atama yapar.
    • Hub Nedir-> Yayın ve haberleşmede kullanılan düşük maliyetli bir cihazdır. Bant genişliğini bütün cihazlara böler, switch’de ise bant genişliğinde azalma olmaz. Ağ içerisinde veri gönderileceği zaman bu veri tüm cihazlara gönderilir.
    • 


Sorular:
    1. A
    2. 
