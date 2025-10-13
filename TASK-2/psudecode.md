BEGIN E-Ticaret_Uygulamasi

  // 1. Kullanıcı Girişi
  DISPLAY "Lütfen kullanıcı adı ve şifrenizi girin."
  INPUT username, password

  IF Authenticate(username, password) == TRUE THEN
    DISPLAY "Giriş başarılı."
  ELSE
    DISPLAY "Giriş başarısız. Lütfen tekrar deneyin."
    EXIT
  ENDIF

  // 2. Ürün Ekleme ve Sepet Yönetimi
  WHILE user_istegi != "alışverişi bitir"
    DISPLAY "Ürün arayın:"
    INPUT arama_terimi

    urun_listesi = UrunAra(arama_terimi)
    DISPLAY urun_listesi

    DISPLAY "Sepete eklemek istediğiniz ürün ID’sini girin:"
    INPUT urun_id

    urun = UrunGetir(urun_id)

    IF StokKontrol(urun) == TRUE THEN
      DISPLAY "Kaç adet eklemek istiyorsunuz?"
      INPUT adet

      IF urun.stok >= adet THEN
        SepeteEkle(urun, adet)
        DISPLAY adet + " adet " + urun.ad + " sepete eklendi."
      ELSE
        DISPLAY "Yetersiz stok. Maksimum " + urun.stok + " adet eklenebilir."
      ENDIF

    ELSE
      DISPLAY "Ürün stokta yok."
    ENDIF

    DISPLAY "Devam etmek için 'evet', alışverişi bitirmek için 'alışverişi bitir' yazın:"
    INPUT user_istegi
  ENDWHILE

  // 3. Sepet Özeti ve İndirim Kodu Uygulama
  DISPLAY "Sepetiniz:"
  SepetiGoster()

  DISPLAY "İndirim kodunuz varsa girin, yoksa ENTER’a basın:"
  INPUT indirim_kodu

  IF indirim_kodu != "" THEN
    IF IndirimKoduGecerliMi(indirim_kodu) == TRUE THEN
      indirim_tutari = IndirimUygula(indirim_kodu)
      DISPLAY "İndirim uygulandı: " + indirim_tutari + "₺"
    ELSE
      DISPLAY "Geçersiz indirim kodu."
    ENDIF
  ENDIF

  // 4. Kargo Hesaplama
  toplam_tutar = SepetToplami() - indirim_tutari
  kargo_ucreti = KargoHesapla(toplam_tutar)

  DISPLAY "Kargo ücreti: " + kargo_ucreti + "₺"

  genel_toplam = toplam_tutar + kargo_ucreti
  DISPLAY "Ödenecek toplam tutar: " + genel_toplam + "₺"

  // 5. Ödeme Aşaması
  DISPLAY "Lütfen ödeme bilgilerinizi girin:"
  INPUT kart_numarasi, son_kullanma, cvc

  IF OdemeYap(kart_numarasi, son_kullanma, cvc, genel_toplam) == TRUE THEN
    DISPLAY "Ödeme başarılı! Siparişiniz hazırlanıyor."
    SiparisOlustur()
  ELSE
    DISPLAY "Ödeme başarısız. Lütfen bilgileri kontrol edin."
  ENDIF

END E-Ticaret_Uygulamasi
