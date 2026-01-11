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

* Sistem Perspektifi – Buffer Pool (Bellek Yönetimi)

Veritabanı sistemlerinde performansı belirleyen en önemli faktörlerden biri disk erişim maliyetidir. Disk erişimi, bellek erişimine kıyasla oldukça yavaştır. Bu nedenle modern veritabanları, sık kullanılan verileri doğrudan diskten okumak yerine RAM üzerinde tutarak performansı artırır. Bu yaklaşım Buffer Pool mekanizması olarak adlandırılır.

PostgreSQL veritabanında disk üzerindeki veriler page (sayfa) kavramı ile yönetilir. PostgreSQL’de varsayılan sayfa boyutu 8 KB’dır ve diskten yapılan okumalar satır bazlı değil, sayfa bazlıdır. Yani tek bir satır istense bile, o satırın bulunduğu tüm sayfa belleğe alınır. Bu sayfa daha sonra Buffer Pool içerisinde cache’lenir.

Buffer Pool’un temel amacı, tekrar eden disk I/O işlemlerini azaltmaktır. Bir sayfa RAM’e alındıktan sonra aynı sayfaya yapılan erişimler doğrudan bellek üzerinden gerçekleşir. Bu durum, sorgu sürelerini ciddi ölçüde düşürür ve sistem genelinde performans artışı sağlar.

PostgreSQL’de Buffer Pool, paylaşımlı bellek alanı olan shared_buffers içerisinde tutulur. Bellek sınırlı olduğu için Buffer Pool dolduğunda hangi sayfanın bellekten çıkarılacağına karar verilmesi gerekir. PostgreSQL bu noktada LRU’ya benzer şekilde çalışan CLOCK-Sweep algoritmasını kullanır. Ancak bu çalışmada özellikle algoritma detaylarından ziyade, verilerin RAM üzerinde cache’lenmesi (bellek yönetimi) konusu ele alınmıştır.

Sonuç olarak Buffer Pool mekanizması, işletim sistemi, disk ve veritabanı arasındaki etkileşimi optimize ederek disk erişimlerini minimize eder ve sistem performansını artırır.

* Veri Yapıları Perspektifi – B+ Tree ve Heap + Index Ayrımı

Veritabanı sistemlerinde veri arama işlemlerinin hızlı olması, kullanılan veri yapıları ile doğrudan ilişkilidir. PostgreSQL’de indeksler temel olarak B+ Tree veri yapısı kullanılarak oluşturulmuştur. B+ Tree, disk tabanlı sistemler için optimize edilmiş, dengeli bir ağaç yapısıdır.

B+ Tree yapısında iç düğümler yalnızca anahtar değerleri ve yönlendirme bilgilerini tutarken, gerçek veriler yaprak (leaf) düğümlerde bulunur. Ayrıca yaprak düğümler birbirine bağlıdır. Bu yapı sayesinde aralık sorguları (range query) oldukça verimli şekilde çalışır.

PostgreSQL’de tablo verileri heap file olarak saklanır. İndeksler ise tablodan tamamen ayrı bir yapıdır. Yani PostgreSQL, heap + index ayrımı yapan bir veritabanıdır. Bir indeks kaydı, anahtar değeri ve bu anahtarın işaret ettiği heap üzerindeki satır adresini (page id + offset) içerir.

Bir sorgu çalıştırıldığında önce B+ Tree indeks üzerinde arama yapılır. Ardından bulunan adres bilgisi kullanılarak heap dosyasındaki gerçek satıra erişilir. Bu yapı, verinin disk üzerinde düzenli ve kontrollü bir şekilde erişilmesini sağlar.

Bu yaklaşım sayesinde PostgreSQL, büyük veri setlerinde dahi O(log n) karmaşıklıkla arama yapabilir. Sonuç olarak B+ Tree ve heap + index ayrımı, PostgreSQL’in ölçeklenebilir ve yüksek performanslı çalışmasının temelini oluşturur.

## VT Üzerinde Gösterilen Kaynak Kodları

Buffer Pool mekanizması için PostgreSQL kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/storage/buffer)) \
B+ Tree indeks yapısı için PostgreSQL nbtree kaynak kodları [Linki]([https://...](https://github.com/postgres/postgres/tree/master/src/backend/access/nbtree)) \


