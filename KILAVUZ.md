# qa-kit kılavuzu

PR'larınızı main'e girmeden önce kontrol eden otomatik bir ekip. Kuralı basit: **işi yapan kendi işine not vermez.**

| Kim | Ne yapar | Ne zaman |
|---|---|---|
| **Okuyucu** | Kodu okur, gerçek hataları satırın altına not düşer (Greptile gibi). | Her değişiklik; GitHub'da sadece o çalışır |
| **Tadımcı** | Kodu hiç görmeden, "ne isteniyordu" listesine göre test yazar ve çalıştırır. | Büyük ve riskli işler, PR'dan önce bilgisayarınızda |
| **Sabotajcı** | İşi bozmaya çalışır: iki kez basma, internet kesilmesi, geri tuşu, küçük ekran… | Büyük ve riskli işler, PR'dan önce bilgisayarınızda |

"Büyük" iş: yeni özellik, mantık değişikliği ya da para, ödeme, hesap gibi hassas dosyalara dokunan her şey. Yazı, renk ve boşluk değişiklikleri küçüktür.

## Kurulum (bir kere)

```bash
claude plugin marketplace add Boby-AI/qa-kit
claude plugin install qa-kit@boby
```

Projenin klasöründe `pnpm install` yaptıysanız göndermeden önceki kontrol de kurulmuştur.

## Günlük kullanım

1. **Kod gönderirken (`git push`)** değişen parçaların hızlı testleri kendiliğinden çalışır. Hata varsa gönderim durur, ekranda nedeni yazar.
2. **PR açmadan önce** Claude'da `/qa-kit:kontrol` yazın. Büyük işte size "ne isteniyordu" listesini gösterir, onaylarsınız; Tadımcı ve Sabotajcı bilgisayarınızda çalışır. Geçmeden Claude PR açmaz. Geçince commit'iniz GitHub'da **qa-kit/local** işaretini alır.
3. **PR açınca** GitHub'da sadece Okuyucu çalışır, birkaç dakikada biter. Taslak (draft) PR'da çalışmaz; **Ready for review** deyince çalışır.

## Raporu okumak

Rapor PR'ın **en son yorumudur**:

- `✅ Passed`: sorun yok.
- `❌ Needs a fix`: altında düzeltilmesi gerekenler, her biri bir satır.
- **Details**: tıklayınca açılır. İçinde test edilen maddeler, küçük öneriler ve **her hata için Claude'a yapıştırılacak hazır düzeltme komutu** vardır.

**Beklemeden başlayın:** Okuyucu ciddi bir hata bulunca raporu beklemez, hemen not düşer.

Okuyucunun notları **Files changed** sekmesinde ilgili satırın altındadır. Her notta **Prompt to fix with AI** açılır kutusu vardır: kopyalayın, Claude'a yapıştırın.

Kurallar:

- Hatayı düzeltmek için **kodu** düzeltin. Tadımcının testlerini (`qa-spec` adlı dosyalar) değiştirmeyin; test yanlış diyorsanız PR'da yazın.
- Her notun başında rengi var: 🔴 **blocker** (gerçek hata, PR'ı durdurur), 🟠 **major** (büyük ihtimalle hata, düzeltin), 🟡 **minor** (öneri, acil değil). Sadece 🔴 PR'ı durdurur.

## Tüm projeyi taramak

Normal kontrol sadece değişen yere bakar. Tüm projeyi taramak için:

Claude'da `/qa-kit:tam-tarama` yazın. Sadece bazı parçalar için adlarını yazın: `/qa-kit:tam-tarama native` ya da `/qa-kit:tam-tarama billing api`.

Parça adları projenin kökündeki `qa-kit.json` dosyasında, `parts` altında yazar.

Büyük bir sürümden önce ya da arada bir "her şey yolunda mı" diye kullanın.

## Sorun olursa

- **Claude PR açmıyor, "/qa-kit:kontrol" diyor:** kontrol bu kodda henüz geçmedi ya da sonra kod değişti. Her şeyi commit'leyip `/qa-kit:kontrol` yazın.
- **Göndermeden önceki kontrol bilgisayarınızda eksik bir araç yüzünden duruyor:** hata mesajında yazan aracı kurun. Acil durumda `git push --no-verify` ile geçebilirsiniz; PR'daki kontrol yine çalışır.
- **Bir projeye eklemek:** projenin köküne `qa-kit.json` ekleyin ([README](README.md#project-config-qa-kitjson)).
