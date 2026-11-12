# VlaamsGoTaxi – Proje durumu (12 Kasım 2026)

## 1. Sistem özeti
- **Site:** Statik site, GitHub Pages (Source: "Deploy from a branch", main /root), alan adı vlaamsgotaxi.be.
- **Backend:** Google Apps Script web uygulaması (Code.gs) + Google Sheet (Chauffeurs, Voertuigen, Reserveringen).
- **Dosyalar:** index.html, bestel-rit-online.html, rezervatie-wijzigen.html, adminpage.html, config.js (VGT_GKEY ve VGT_SCRIPT_URL; üzerine yazılmamalı).
- **Script Properties:** ADMIN_PIN, OWNER_EMAIL, SHEET_ID (yedek), DISCOUNT, TWILIO_SID, TWILIO_TOKEN, TWILIO_FROM.
- **Güncelleme kuralı:** Code.gs değişince Apps Script'te "Implementeren > Implementaties beheren > düzenle > Nieuwe versie". Sayfa dosyaları değişince sadece o dosyayı GitHub'a yükleyin; config.js'e dokunmayın.

## 2. Şu ana kadar yapılanlar
- E-postaların ve eski rezervasyonların görünmemesi çözüldü (site yanlış Apps Script adresine bakıyordu).
- Şoför kodları isme göre: ilk harf + soyad (örn. syelkenci); aynı kod çıkarsa 2, 3… eklenir.
- İndirim yüzdesi admin sayfasındaki Instellingen sekmesinden değiştirilir (kodda gömülü değil).
- QR kodu kopyalama biçimi: `Code,Naam`.
- QR kodu telefonda tanınmama sorunu çözüldü (GitHub Pages yayın kaynağı yanlıştı).
- Admin sayfası: tarih ön ayarları (bugün/hafta/ay…), şoför ve fiyat aralığı filtreleri, mobil kart görünümü, hızlı kaydet/sil.
- Rezervasyon sayfası 3 sekmeli: Uw rit, Uw gegevens, Betaalmethode.
  - Ödeme yöntemleri ayar listesinden genişletilebilir (Bancontact vb. için altyapı hazır, şimdilik kapalı).
  - 30 km üstü ritlerde "betaling vooraf" mantığı var.
  - Sekmeler sırayla açılır: önceki adım geçerli değilse sonraki kilitli.
  - Masaüstünde form ve harita eşit yükseklikte, sayfa kaydırması gerekmez. Rota çizilince harita rotaya sığacak şekilde yakınlaşır.
  - Telefonda harita formun en altında.
  - İkonlar ince çizgili ve sade.
- Korting kodu silinince fiyat kutusunda kalan "Gefeliciteerd" mesajı düzeltildi.
- **SMS (kod hazır):**
  - Müşteri "Bevestiging via" seçer: e-posta (varsayılan), SMS veya ikisi.
  - SMS türleri: rezervasyon onayı, hatırlatma (~2 saat önce), değişiklik/iptal onayı.
  - Sadece Belçika (04…) ve Hollanda (06…) cep numaraları kabul edilir.
  - Günlük limit: toplam 60, numara başına 6 (Script Properties ile değişir).
  - SMS gitmezse ve e-posta varsa onay e-postayla gider.
  - Sheet'e `Melding` ve `Herinnering` kolonları eklenir.
  - Admin sayfasında SMS seçimi ve hatırlatma durumu görünür.

## 3. Şu an bekleyen / sırada ne var
Twilio hesabı yükseltildi ve bakiye yüklendi, ama SMS henüz gitmiyor.

| Durum | Not |
|---|---|
| Hata 572006 (deneme hesabı özel metin gönderemez) | Hesap yükseltilerek aşıldı. |
| Hata 20003 "Primary compliance profile is not approved" | **Açık.** Twilio Trust Hub'da KYC bitmeli. |
| Alan adı doğrulaması | Twilio'nun verdiği TXT kaydı (`_twilio` host) DNS'e eklenmeli, sonra "Check verification". Yayılma en fazla 72 saat. |

Sıradaki adımlar:
1. DNS'e TXT kaydını ekleyin (GoDaddy veya DNS'i nerede yönetiyorsanız): Tür TXT, Host `_twilio`, Değer Twilio ekranındaki `twilio-domain-verification=…` satırı. Sonra Twilio'da "Check verification".
2. Trust Hub > Customer Profiles > Primary Customer Profile'ı doldurup onaya gönderin (firma adı, adres, KBO/BTW numarası, yetkili kişi).
3. Twilio onayı gelince Apps Script'te `testSms` çalıştırın (numara satırı: "0474 64 07 88"). Log "SMS verstuurd" yazmalı.
4. Çalışırsa `TWILIO_FROM` değerini `VlaamsGoTaxi` yapıp tekrar deneyin. Hata verirse Twilio numarasına (+4915888620339) geri dönün.
5. `installReminderTrigger` fonksiyonunu bir kez çalıştırın (hatırlatmalar için).
6. Sitede SMS seçip gerçek bir deneme rezervasyonu yapın; onay SMS'ini, değişiklik/iptal SMS'ini ve hatırlatmayı kontrol edin.
7. KYC çok uzarsa alternatif: Belçika'ya SMS gönderen başka bir sağlayıcı (sadece `sendSms` fonksiyonu değişir).

## 4. Kontrol listesi (en son sürüm yayında mı?)
- [ ] Code.gs son sürüm Apps Script'te, "Nieuwe versie" yayınlandı, `authorize` çalıştırıldı.
- [ ] bestel-rit-online.html (PAGE_BUILD 2026-11-11-b) ve adminpage.html GitHub'da.
- [ ] Telefonda sayfa yenilenince harita formun altında, SMS seçeneği görünüyor.

## 5. Opsiyonel fikirler (istenmedi, sonra bakılabilir)
- Telefon numarasını config.js'e taşımak (`VGT_TEL`).
- Admin sayfasında "Bugünün yolculukları" kısayolu.
- 4 haneli rezervasyon numaraları.
- Online ödeme (Bancontact) altyapısı hazır; ödeme sağlayıcısı bağlanınca açılır. Şimdilik beklemede.
- Belçika'da SMS için müşteri izni konusunda bir hukuk danışmanına sormak iyi olur.
