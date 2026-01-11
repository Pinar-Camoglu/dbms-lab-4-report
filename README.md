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

Bu çalışmada, veritabanı sistemlerinde performansı etkileyen temel unsurlar sistem programlama ve veri yapıları bakış açısıyla ele alınmıştır. Günlük hayatta kullanılan birçok yazılımın arka planında veritabanları yer almakta ve bu sistemlerin hızlı çalışması büyük önem taşımaktadır. Bu nedenle, veritabanlarının disk ve bellek gibi donanım kaynaklarını nasıl kullandığı incelenmiş; açık kaynak kodlu bir veritabanı olan PostgreSQL üzerinden örnekler verilmiştir. Çalışmada Sistem Perspektifi kapsamında Buffer Pool mekanizması, Veri Yapıları Perspektifi kapsamında ise B+ Tree indeks yapısı ve PostgreSQL’in heap + index ayrımı ele alınmıştır.

Veritabanı sistemlerinde en maliyetli işlemlerden biri disk erişimidir. Diskten veri okumak, belleğe göre çok daha yavaş olduğu için performans üzerinde doğrudan etkilidir. Bu nedenle modern veritabanları, sık kullanılan verileri her seferinde diskten okumak yerine bellekte tutmayı tercih eder. PostgreSQL’de bu görev Buffer Pool mekanizması tarafından gerçekleştirilir. Disk üzerindeki veriler PostgreSQL’de satır bazlı değil, sayfa bazlı olarak yönetilir. Varsayılan olarak her sayfa 8 KB boyutundadır ve diskten yapılan okuma işlemleri bu sayfalar üzerinden yapılır.

Buffer Pool, diskten okunan bu sayfaların RAM üzerinde saklanmasını sağlar. Bir sayfa belleğe alındıktan sonra, aynı sayfaya tekrar erişilmek istendiğinde disk yerine doğrudan bellek kullanılır. Bu durum, özellikle sık tekrar eden sorguların çalıştığı sistemlerde önemli bir performans artışı sağlar. PostgreSQL’de Buffer Pool, shared_buffers adı verilen paylaşımlı bellek alanı içerisinde tutulur. Bu yapı sayesinde birden fazla işlem, aynı sayfayı tekrar tekrar diskten okumak zorunda kalmaz.

Bellek sınırlı bir kaynak olduğu için, Buffer Pool’un tamamen dolması durumunda hangi sayfanın bellekten çıkarılacağına karar verilmesi gerekir. PostgreSQL bu noktada, LRU mantığına benzer şekilde çalışan CLOCK-Sweep algoritmasını kullanır. Bu algoritma, uzun süre kullanılmayan sayfaların bellekten çıkarılmasını sağlar. Ancak bu çalışmada özellikle algoritmanın detaylarından ziyade, veritabanının verileri RAM üzerinde cache’leyerek disk erişimlerini azaltma yaklaşımı üzerinde durulmuştur.

Çalışmanın ikinci bölümünde veri yapıları perspektifinden B+ Tree indeks yapısı incelenmiştir. PostgreSQL’de indeksler, disk tabanlı sistemler için uygun olan B+ Tree veri yapısı kullanılarak oluşturulmaktadır. B+ Tree yapısında iç düğümler yalnızca anahtar değerleri ve yönlendirme bilgilerini tutarken, asıl veriler yaprak düğümlerde saklanır. Ayrıca yaprak düğümlerin birbirine bağlı olması, aralık sorgularının (range query) hızlı bir şekilde çalışmasını sağlar.

PostgreSQL’de tablo verileri heap file olarak saklanır ve indekslerden ayrı bir yapıdadır. Bu nedenle PostgreSQL, heap + index ayrımı yapan bir veritabanı olarak tanımlanır. İndeksler, satırların kendisini değil; bu satırların heap üzerindeki konumunu gösteren adres bilgilerini tutar. Bir sorgu çalıştırıldığında önce indeks üzerinde arama yapılır, ardından elde edilen adres bilgisi kullanılarak heap dosyasındaki gerçek satıra erişilir.

Sonuç olarak Buffer Pool mekanizması ve B+ Tree indeks yapısı, PostgreSQL’in hem okuma hem de yazma işlemlerinde dengeli ve yüksek performanslı çalışmasını sağlar. Disk erişimlerinin azaltılması, belleğin verimli kullanılması ve uygun veri yapılarının tercih edilmesi sayesinde PostgreSQL, büyük veri kümeleri üzerinde dahi etkili bir şekilde çalışabilmektedir.

## VT Üzerinde Gösterilen Kaynak Kodları

Buffer Pool mekanizması için PostgreSQL kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/storage/buffer)) \
B+ Tree indeks yapısı için PostgreSQL nbtree kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/access/nbtree)) \


