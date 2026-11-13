# Nova AI — Cloudflare Access ile e-posta girişi

Bu uygulamada isteğe bağlı Cloudflare Access **tek kullanımlık e-posta koduyla giriş (OTP)** altyapısı hazırdır.

> Bu, Gmail/Outlook gelen kutusu bağlama veya e-posta gönderme özelliği değildir.
> Cloudflare Access, kimlik doğrulamasını kendi sayfasında gerçekleştirir.
> Nova AI asla doğrulama kodunu veya e-posta şifrenizi istemez.

## Ön koşullar

- Cloudflare DNS üzerinde **size ait bir alan adı** (ör. `nova.ornekalanadi.com`).
- Cloudflare Zero Trust / Access.
- Render'daki mevcut Nova AI web servisi.
- Cloudflare hesabında e-posta kodu (One-time PIN) kimlik sağlayıcısı.

## 1) Alan adını Render'a bağlayın

Render > `nova-ai-groq-gemini` > Settings > Custom Domains bölümünden alan adını ekleyin.
Cloudflare DNS'te (alan adı sahibine ait) uygun bir **proxied** CNAME kaydı oluşturun:

- Host: `nova` (örnek)
- Hedef: `nova-ai-groq-gemini.onrender.com`
- Proxy: **Proxied (orange cloud)**

Render > Custom Domains ekranında **Verify** ile alan adının bağlandığını doğrulayın.

## 2) Cloudflare Access ile e-posta kodlu giriş

Cloudflare One / Zero Trust > Access controls > Applications:
1. **Create new application** > **Self-hosted**.
2. Public hostname olarak az önce bağladığınız `nova.ornekalanadi.com` adresini seçin.
3. Access policy ile yalnızca izin vereceğiniz e-posta adreslerini tanımlayın.
4. Login method olarak **One-time PIN** etkinleştirin.
5. Uygulamanın **Application Audience (AUD)** değerini kopyalayın.
6. Takım alan adınızı öğrenin: `https://TAKIM_ADI.cloudflareaccess.com`

Önemli: Erişim politikasını `Everyone` yapmak, herkese e-posta koduyla giriş hakkı verebilir. İzinli e-posta adresleri tanımlayın.

## 3) Render'da JWT doğrulamasını açın

Render > `nova-ai-groq-gemini` > Environment bölümünde iki değişken ekleyin:

```dotenv
CF_ACCESS_TEAM_DOMAIN=https://TAKIM_ADI.cloudflareaccess.com
CF_ACCESS_AUD=UYGULAMA_AUDIENCE_DEGERI
```

Bunlar eklenmedikçe Cloudflare giriş sistemi **pasiftir** ve mevcut site çalışmaya devam eder.

**İki değişkeni de ancak Cloudflare Access ve alan adı gerçekten çalıştıktan sonra ekleyin.** Etkinleştirildiğinde Nova AI sunucusu `/api/chat` isteklerinde Cloudflare Access JWT imzasını, süresini, audience ve issuer değerlerini denetler. Böylece `onrender.com` üzerinden gelen sahte girişler kabul edilmez; doğrudan bu alan adından AI isteği göndermek engellenir.

## 4) Test

1. Yeni özel alan adınızdan Nova AI'yi açın; Cloudflare Access giriş ekranı çıkmalı.
2. İzinli e-postanızı girip kodu e-postanızdan alın.
3. Kodla giriş yapın.
4. Nova AI üst çubuktaki **E-posta** bölümünden doğrulanmış e-posta adresinizi göstermeli.
5. Çıkış için **Çıkış yap** düğmesini kullanabilirsiniz.

## Sınırlamalar

- **Sohbetler şu anda sadece kullanılan tarayıcının yerel depolamasındadır.** E-posta ile giriş yapmak otomatik olarak bulut yedekleme veya cihazlar arası senkronizasyon sağlamaz.
- Kullanıcıların gerçek e-postalarını hiçbir zaman kod deposuna veya istemciye sabit değer olarak yazmayın.
- `CF_ACCESS_TEAM_DOMAIN` ve `CF_ACCESS_AUD` değerlerini girmeden site e-posta ile giriş ekranına geçmez.
- Cloudflare, e-posta ile giriş kodlarını kendi servisinden gönderir; Render'da ayrıca SMTP şifresine gerek yoktur.
- AI API anahtarları Render ortam değişkenlerinde kalmalıdır.

## Resmî kaynaklar

- [Cloudflare Access: One-time PIN](https://developers.cloudflare.com/cloudflare-one/integrations/identity-providers/one-time-pin/)
- [Cloudflare Access: Self-hosted application](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/self-hosted-public-app/)
- [Cloudflare Access: Validate JWTs](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/authorization-cookie/validating-json/)
- [Render: Custom Domains](https://render.com/docs/custom-domains)
- [Render: Configure Cloudflare DNS](https://render.com/docs/configure-cloudflare-dns)
