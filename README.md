# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [ ]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [ ]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [ ]  LRU / CLOCK gibi algoritmaları
- [ ]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [X]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki](https://www.youtube.com/watch?v=Nw1OvCtKPII&t=2635s) 
Ekran kaydı. 2-3 dk. açık kaynak V.T. kodu üzerinde konunun gösterimi. Video kendini tanıtma ile başlamalıdır (Numara, İsim, Soyisim, Teknik İlgi Alanları). 

---

# Açıklama
Bir Bilgisayar Mühendisliği öğrencisi olarak veritabanı sistemlerine baktığımda, bu yapıların sadece veri saklayan depolar 
olmadığını; aksine Sistem Programlama ve Veri Yapıları derslerinde gördüğümüz teorik kavramların en uç noktada optimize 
edilerek kullanıldığı karmaşık yazılımlar olduğunu görüyorum. Bu deney çalışmasında, açık kaynak kodlu PostgreSQL veritabanı 
yönetim sistemi üzerinde, performansı doğrudan etkileyen iki kritik bileşeni inceledim: Disk I/O yönetimini sağlayan Buffer 
Pool mekanizması ve veriye erişim stratejisini belirleyen Heap ile Index (B+ Tree) ayrımı.

1- Sistem Programlama Perspektifi: Disk Erişimi ve Buffer Pool Modern bilgisayar mimarilerinde işlemci (CPU) ve ana bellek
(RAM) hızları nanosaniyelerle ölçülürken, disk erişim süreleri (SSD olsa dahi) mikrosaniyeler veya milisaniyeler
seviyesindedir. Bu durum, sistem programlamada "I/O Darboğazı" (I/O Bottleneck) olarak adlandırılır. Bir veritabanı, her
sorgu için diske fiziksel erişim (syscall: read) yapmak zorunda kalsaydı, sistem kaynakları verimsiz kullanılır ve yanıt
süreleri kabul edilemez seviyelere çıkardı.

PostgreSQL, bu donanımsal gecikmeyi maskelemek için işletim sisteminin dosya önbelleğinden (OS Page Cache) bağımsız, kendi 
yönettiği bir bellek alanı olan "Shared Buffers" (Buffer Pool) mekanizmasını kullanır. Veritabanı, verileri diskten "Page" 
(Sayfa) adı verilen ve varsayılan boyutu 8 KB olan bloklar halinde okur. Bir sorgu çalıştırıldığında, PostgreSQL motoru 
istenen verinin bulunduğu sayfanın halihazırda Buffer Pool’da olup olmadığını kontrol eder. Eğer sayfa bellekteyse (Cache 
Hit), disk erişimi maliyeti sıfırlanır ve veri doğrudan bellekten sunulur. Eğer değilse (Cache Miss), sayfa diskten belleğe 
kopyalanır.

Ancak RAM sonsuz bir kaynak değildir. Bellek dolduğunda, yeni verilere yer açmak için eski sayfaların tahliye edilmesi 
(Eviction) gerekir. PostgreSQL burada, Sistem Programlama derslerinde gördüğümüz LRU (Least Recently Used) algoritmasının, 
yüksek eşzamanlılık (concurrency) için optimize edilmiş bir versiyonu olan Clock Sweep (Saat) algoritmasını kullanır. Bu 
algoritma, sayfaların kullanım sıklığını (usage count) takip ederek, sık kullanılan "sıcak" verilerin bellekte kalmasını 
garanti altına alır. Bu sayede veritabanı, donanımın fiziksel limitlerini akıllı yazılım mimarisiyle aşar.

2- Veri Yapıları Perspektifi: B+ Tree ve Heap Ayrımı Verinin belleğe alınması performansı artırır, ancak verinin bellekte
veya diskte "nasıl" bulunduğu ve arandığı Veri Yapıları dersinin konusudur. PostgreSQL’i diğer birçok veritabanından ayıran
en temel özellik, Heap (Yığın) ve Index (İndeks) yapılarını birbirinden kesin çizgilerle ayırmasıdır.

PostgreSQL’de tablo verileri (satırların kendisi), "Heap" adı verilen ve verilerin ekleme sırasına göre düzensiz bir 
şekilde yığıldığı dosyalarda saklanır. Heap yapısında belirli bir veriyi bulmanın maliyeti $O(N)$ karmaşıklığındadır, yani 
tüm dosyanın taranması gerekir (Sequential Scan). Bu verimsizliği aşmak için B+ Tree veri yapısı devreye girer. Disk 
tabanlı sistemler için optimize edilmiş, çok dallı (high fan-out) ve dengeli bir ağaç yapısı olan B+ Tree, arama maliyetini 
$O(\log N)$ seviyesine indirir.

Buradaki kritik nokta şudur: PostgreSQL’de B+ Tree indeksleri, verinin kendisini barındırmaz (Non-Clustered). İndeks 
yaprağında sadece arama anahtarı (Key) ve o verinin Heap içindeki fiziksel konumunu gösteren bir pointer (TID - Tuple 
Identifier) bulunur. Bir sorgu geldiğinde, veritabanı önce B+ Tree üzerinde arama yaparak ilgili anahtarın adresini (Block 
Number + Offset) bulur, ardından bu adresi kullanarak Heap dosyasındaki asıl veriye erişir. Bu "Heap + Index" ayrımı, 
özellikle ikincil indekslerin (secondary indexes) oluşturulmasını çok ucuzlatır çünkü ana veri (Heap) yer değiştirdiğinde 
veya güncellendiğinde, verinin taşınmasına gerek kalmaz, sadece pointerlar güncellenir.

Sonuç Özetle; PostgreSQL’in yüksek performansı tesadüf değildir. Sistem perspektifinde, Buffer Pool sayesinde disk 
yavaşlığı elimine edilir ve I/O minimize edilir. Veri yapıları perspektifinde ise B+ Tree ve Heap ayrımı sayesinde, 
milyonlarca satır içinden aranan veri, minimum sayıda işlemle (pointer takibiyle) bulunur. Bu deney, teorik derslerde 
gördüğümüz kavramların gerçek dünyadaki en güçlü uygulamalarından birini analiz etmemi sağlamıştır.

## VT Üzerinde Gösterilen Kaynak Kodları

Buffer Pool mekanizması için PostgreSQL kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/storage/buffer)) \
B+ Tree indeks yapısı için PostgreSQL nbtree kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/access/nbtree)) \


