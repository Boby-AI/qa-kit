# qa-kit kılavuzu

PR'larınızı main'e girmeden önce kontrol eden otomatik bir ekip. Kuralı basit: **işi yapan kendi işine not vermez.**

| Kim | Ne yapar | Ne zaman |
|---|---|---|
| **Okuyucu** | Kodu okur, gerçek hataları satırın altına not düşer (Greptile gibi). | Her PR'da GitHub'da, birkaç dakikada |
| **Tadımcı** | Kodu hiç görmeden, "ne isteniyordu" listesine göre test yazar ve çalıştırır. | Sadece siz isteyince: `/qa-kit:kontrol derin` |
| **Sabotajcı** | İşi bozmaya çalışır: iki kez basma, internet kesilmesi, geri tuşu, küçük ekran… | Sadece siz isteyince: `/qa-kit:kontrol derin` |

Tadımcı ve Sabotajcı yavaş ve pahalıdır; ödeme, kredi, hesap gibi riskli bir işte siz isteyince çalışırlar. Günlük işte hiçbir şey beklemeniz gerekmez.

## Kurulum (bir kere)

```bash
claude plugin marketplace add Boby-AI/qa-kit
claude plugin install qa-kit@boby
```

Projenin klasöründe `pnpm install` yaptıysanız göndermeden önceki kontrol de kurulmuştur.

## Günlük kullanım

1. **Kod gönderirken (`git push`)** değişen parçaların hızlı testleri kendiliğinden çalışır. Hata varsa gönderim durur, ekranda nedeni yazar.
2. **İsterseniz PR'dan önce** `/qa-kit:kontrol` yazın: hızlı testler ve Okuyucu, birkaç dakika. Riskli bir işte `/qa-kit:kontrol derin` yazın: size kısa bir "ne isteniyordu" özeti gösterir, onaylarsınız; Tadımcı ve Sabotajcı da çalışır (daha uzun sürer). Hiçbiri zorunlu değil.
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

- **Claude PR açmadan önce kontrol çalıştırıyor:** kontrol bu kodda henüz geçmedi ya da sonra kod değişti. Beklemeniz yeterli.
- **Göndermeden önceki kontrol bilgisayarınızda eksik bir araç yüzünden duruyor:** hata mesajında yazan aracı kurun. Acil durumda `git push --no-verify` ile geçebilirsiniz; PR'daki kontrol yine çalışır.
- **Bir projeye eklemek:** projenin köküne `qa-kit.json` ekleyin ([README](README.md#project-config-qa-kitjson)).
