## Online Görüntüleme (Public URL)

Bu uygulamayı online yapmanın 2 pratik yolu var:

- **Kalıcı (önerilen)**: Render gibi bir servise deploy → sabit URL
- **Hızlı/geçici**: Bilgisayarınızdan tünel aç → anında public link (PC açık kalmalı)

### Seçenek A: Render ile kalıcı deploy (sabit URL)

1) Projeyi bir GitHub reposuna koyun (`qr-uygulama` klasörü repo kökü olabilir).

2) Render’da **New → Web Service** oluşturun, repo’yu seçin.

3) Ayarlar:
- **Build Command**: `pip install -r requirements.txt`
- **Start Command**: `gunicorn -w 2 -b 0.0.0.0:$PORT wsgi:app`

4) Deploy olduktan sonra size bir URL verir (ör. `https://...onrender.com`).

5) **Kalıcı kayıt için (önemli)** Render’da servise **Persistent Disk** ekleyin:
- **Mount Path**: `/var/data`

6) Render → **Environment** değişkenleri ekleyin:
- **QR_CONFIG_PATH**: `/var/data/config.json`  (ayarlar diske yazılsın)
- **ADMIN_TOKEN**: güçlü bir değer belirleyin (ör. 20+ karakter)  (admin’e sizin bildiğiniz token ile girin)
- **APP_MODE**: `host_only`  (Render sadece /info host etsin, QR üretmesin)

7) Admin’e girip bilgileri düzenleyin:
- `https://SIZIN-URL/admin?token=ADMIN_TOKEN`

> Not: Disk eklemezseniz, bazı platformlarda dosya değişiklikleri deploy/restart sonrası kaybolabilir. Disk kullanmak bu problemi çözer. İsterseniz daha da sağlam olsun diye DB (Supabase/Postgres) seçeneğini de ekleyebilirim.

### “QR lokal, bilgi sayfası online” çalışma şekli

- Müşteriler: QR okuttuğunda önce Render host’a gider, host da **statik siteye yönlendirir**.
- Siz (kendi PC’nizden):
  - `config.json` → `public_base_url` ve `remote_base_url` içine `https://SIZIN-URL` (Render host) yazın
  - `remote_admin_token` içine Render’daki `ADMIN_TOKEN` değerini yazın
  - `static_redirect_url` içine statik sitenizi yazın (ör. `https://statik-qr-website.onrender.com`)
  - `remote_rotate_enabled: true` yapın (eski QR'lar geçersiz olsun)
  - Metni güncelledikten sonra: `python sync_remote.py` (host'a gönderir)

QR içeriği şu formatta olur:
- `https://SIZIN-URL/r/<token>`
Host sadece **en son token** ile gelen istekleri statik siteye yönlendirir; eski QR tokenları **410 Gone** alır.

### Seçenek B: Cloudflare Tunnel (hızlı public link)

Bu yöntemle uygulama **sizin bilgisayarınızda** çalışır; Cloudflare public URL verir.

1) `cloudflared` kurun.
2) Uygulamayı lokal çalıştırın:
   - `python app.py`
3) Yeni bir terminalde tünel açın:
   - `cloudflared tunnel --url http://127.0.0.1:8000`

Çıktıda bir `https://....trycloudflare.com` linki göreceksiniz. QR artık bu link üzerinden açılır.

### Seçenek C: ngrok (hızlı public link)

1) ngrok kurun ve token’ınızı ayarlayın.
2) Uygulamayı lokal çalıştırın:
   - `python app.py`
3) Tünel:
   - `ngrok http 8000`

ngrok size bir `https://...ngrok-free.app` gibi URL verir.


