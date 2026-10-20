# qa-kit kılavuzu

PR'larınızı main'e girmeden önce kontrol eden otomatik bir ekip. Kuralı basit: **işi yapan kendi işine not vermez.**

| Kim | Ne yapar | Ne zaman |
|---|---|---|
| **Okuyucu** | Kodu okur, gerçek hataları satırın altına not düşer (Greptile gibi). | Her PR |
| **Tadımcı** | Kodu hiç görmeden, "ne isteniyordu" listesine göre test yazar ve çalıştırır. | Büyük ve riskli işler |
| **Sabotajcı** | İşi bozmaya çalışır: iki kez basma, internet kesilmesi, geri tuşu, küçük ekran… | Büyük ve riskli işler |

"Büyük" iş: yeni özellik, mantık değişikliği ya da para, ödeme, hesap gibi hassas dosyalara dokunan her şey. Yazı, renk ve boşluk değişiklikleri küçüktür.

## Kurulum (bir kere)

```bash
claude plugin marketplace add Boby-AI/qa-kit
claude plugin install qa-kit@boby
```

Projenin klasöründe `pnpm install` yaptıysanız göndermeden önceki kontrol de kurulmuştur.

## Günlük kullanım

1. **Kod gönderirken (`git push`)** değişen parçaların hızlı testleri kendiliğinden çalışır. Hata varsa gönderim durur, ekranda nedeni yazar.
2. **PR açınca** bir şey yapmanıza gerek yok. Taslak (draft) PR'da çalışmaz; **Ready for review** deyince çalışır. Mevcut testlerle aynı anda yürür.
3. **İsterseniz önce kendiniz:** Claude'da `/qa-kit:kontrol` yazın. Büyük işte size "ne isteniyordu" listesini gösterir; onaylarsanız PR o listeyle test edilir. Yazmazsanız GitHub listeyi PR açıklamasından kendisi çıkarır.

## Raporu okumak

Rapor PR'ın **en son yorumudur**:

- `✅ Passed`: sorun yok.
- `❌ Needs a fix`: altında düzeltilmesi gerekenler, her biri bir satır.
- **Details**: tıklayınca açılır. İçinde test edilen maddeler, küçük öneriler ve **her hata için Claude'a yapıştırılacak hazır düzeltme komutu** vardır.

Okuyucunun notları **Files changed** sekmesinde ilgili satırın altındadır. Her notta **Prompt to fix with AI** açılır kutusu vardır: kopyalayın, Claude'a yapıştırın.

Kurallar:

- Hatayı düzeltmek için **kodu** düzeltin. Tadımcının testlerini (`qa-spec` adlı dosyalar) değiştirmeyin; test yanlış diyorsanız PR'da yazın.
- Notların hepsi zorunlu değil. Sadece gerçek hatalar (blocker) PR'ı durdurur, gerisi öneridir.

## Tüm projeyi taramak

Normal kontrol sadece değişen yere bakar. Tüm projeyi taramak için:

- **GitHub'dan:** repoda **Actions** → soldan **qa-kit** → **Run workflow**. Bir parça adı yazabilir (ör. `native`) ya da boş bırakıp her yeri taratabilirsiniz. 20-40 dakika sürer; rapor o çalıştırmanın **Summary** sayfasındadır.
- **Bilgisayarınızdan:** Claude'da `/qa-kit:tam-tarama`. Sadece bazı parçalar için adlarını yazın: `/qa-kit:tam-tarama native` ya da `/qa-kit:tam-tarama billing api`.

Parça adları projenin kökündeki `qa-kit.json` dosyasında, `parts` altında yazar.

Büyük bir sürümden önce ya da arada bir "her şey yolunda mı" diye kullanın.

## Sorun olursa

- **"spec" ya da liste ile ilgili bir şey soruyor:** `/qa-kit:kontrol` yazıp gösterdiği listeyi onaylayın.
- **Göndermeden önceki kontrol bilgisayarınızda eksik bir araç yüzünden duruyor:** hata mesajında yazan aracı kurun. Acil durumda `git push --no-verify` ile geçebilirsiniz; PR'daki kontrol yine çalışır.
- **Bir projeye eklemek:** projenin köküne `qa-kit.json` ekleyin ([README](README.md#project-config-qa-kitjson)).
