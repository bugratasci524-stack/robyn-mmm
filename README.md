# Robyn MMM örnek çalışma

Meta Robyn'in Python sürümü (robynpy) ile örnek veri seti üzerinde uçtan uca Marketing Mix Modeling (MMM) akışı: veri hazırlığı, model eğitimi, model seçimi, one-pager ve bütçe optimizasyonu.

> **Not:** Bu repo öğrenme amaçlıdır. robynpy 0.3.6 beta sürümünde, eğitim sonrası ayrıştırmada tahminlerin seviye olarak aşağı kaydığı bir hata tespit edildi (bkz. Bilinen sorunlar). One-pager ve bütçe optimizasyonundaki rakamlar gerçek bir çalışmada kullanılmamalıdır.

## Veri

Veri dosyaları repoda tutulmuyor. İki dosyayı indirip proje klasörüne kaydedin:

- [dt_simulated_weekly.csv](https://raw.githubusercontent.com/facebookexperimental/Robyn/main/python/src/robyn/tutorials/resources/dt_simulated_weekly.csv): 208 haftalık simüle veri (2015-11 ile 2019-11 arası)
- [dt_prophet_holidays.csv](https://raw.githubusercontent.com/facebookexperimental/Robyn/main/python/src/robyn/tutorials/resources/dt_prophet_holidays.csv): ülke bazlı tatil tablosu

CSV dosyalarını Excel'de açıp kaydetmeyin. Türkçe ayarlı Excel tarih biçimini ve ayırıcıları değiştiriyor, Robyn dosyayı okuyamıyor.

| Kolon | Modeldeki rolü |
|---|---|
| revenue | Bağımlı değişken (ciro) |
| tv_S, ooh_S, print_S, facebook_S, search_S | Ücretli medya harcamaları |
| facebook_I, search_clicks_P | Facebook ve Search için maruziyet metrikleri |
| newsletter | Organik kanal |
| competitor_sales_B, events | Kontrol değişkenleri |

## Kurulum (Windows)

robynpy, ridge regresyon adımında R'daki glmnet paketini kullanıyor. Bu yüzden Python ile çalışılsa da R kurulu olmalı. RStudio gerekmiyor.

**1. R ve glmnet**

[cloud.r-project.org](https://cloud.r-project.org) adresinden R'ı kurun. R konsolunda:

```r
install.packages("glmnet", repos = "https://cloud.r-project.org")
R.home()
```

`R.home()` çıktısındaki klasör yolunu not edin.

**2. Python ortamı (PowerShell)**

```powershell
python -m venv robyn-env
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
robyn-env\Scripts\Activate.ps1
$env:R_HOME = "R kurulum klasörü"
$env:PATH = "$env:R_HOME\bin\x64;$env:PATH"
$env:RPY2_CFFI_MODE = "ABI"
pip install -r requirements.txt
```

`RPY2_CFFI_MODE=ABI` ayarı, rpy2'nin Windows'ta derleyici gerektirmeden kurulmasını sağlıyor. Sürümler `requirements.txt` içinde sabitlendi; `pip install robynpy` tek başına pandas 3 ve plotly 6 getiriyor ve ikisi de akışı bozuyor.

**3. Notebook**

VS Code'da Python ve Jupyter eklentilerini kurun, `robyn_model.ipynb` dosyasını açın ve kernel olarak `robyn-env` ortamını seçin. İlk hücredeki `R_HOME` yolunu kendi R kurulumunuza göre güncelleyin.

## Notebook akışı

| Hücre | Adım | Süre |
|---|---|---|
| 1 | R bağlantısı (R_HOME, DLL klasörü, glmnet kontrolü) | Saniyeler |
| 2 | Veri ve tatil tablosunu yükleme | Saniyeler |
| 3 | Değişken tanımları ve modelleme penceresi (2016-2018) | Saniyeler |
| 4 | Trend, sezon ve tatil ayrıştırması (Prophet, ülke: DE) | Saniyeler |
| 5 | Hiperparametre aralıkları (adstock, doygunluk, lambda, train_size) | Saniyeler |
| 6 | Başlatma ve özellik mühendisliği | Saniyeler |
| 7 | Eğitim: 2000 iterasyon x 5 deneme | 5-7 dakika |
| 8 | Yakınsama mesajları ve grafikleri, output klasörüne kayıt | Saniyeler |
| 9 | Pareto seçimi ve kümeleme | Yaklaşık 1 dakika |
| 10 | Aday modeller tablosu | Saniyeler |
| 11 | Seçilen modelin one-pager'ı, output klasörüne kayıt | Saniyeler |
| 12 | Bütçe optimizasyonu | Saniyeler |
| 13 | Bütçe sonucu tablosu | Saniyeler |

"Run All" yerine hücreleri sırayla çalıştırın. Her eğitimde farklı modeller çıktığı için Hücre 11 ve 12'deki model kimliği (`sol_id`) Hücre 10'daki tabloya göre güncellenmeli.

## Model seçim kriterleri

| Metrik | İyi olan |
|---|---|
| rsq_train, rsq_test | Yüksek (1'e yakın) |
| nrmse_train, nrmse_test | Düşük (0'a yakın) |
| decomp.rssd | Düşük (kanal katkıları harcama paylarıyla tutarlı) |

Tek tek sütunlardaki binde birlik farklardan çok, train ile test arasındaki farka bakın. Test hatası eğitim hatasının belirgin şekilde üstündeyse model ezberlemiş demektir.

## Bilinen sorunlar

- **Ayrıştırma hatası (robynpy 0.3.6):** Eğitim sonrası ayrıştırmada tahmin edilen ciro gerçeğin sistematik olarak altında kalıyor, oysa raporlanan NRMSE ve R² değerleri iyi. One-pager'daki katkı payları, ROAS'lar, tepki eğrileri ve bütçe optimizasyonu bu ayrıştırmadan hesaplandığı için güvenilir değil.
- **Yakınsama eşiği:** "NOT converged" mesajındaki medyan eşiği negatif çıkabiliyor ve bu durumda koşul hiçbir zaman sağlanamıyor. Dağılımın daralıp daralmadığına ve hata seviyesine bakın.
- **Grafikler dosyaya yazılmıyor:** Python sürümü yakınsama grafiklerini ve one-pager'ı otomatik kaydetmiyor. Hücre 8 ve 11 bunları output klasörüne elle kaydediyor.
- **Bütçe optimizasyonunda takılma:** Grafik kaydetme aracı (kaleido) Windows'ta takılabiliyor. Hücre 12'de `plots=False` ve `display_plots=False` kullanılıyor, sonuç Hücre 13'teki tablodan okunuyor.
- **Kernel yeniden başlarsa:** Eğitilmiş model hafızadan silinir. Hücre 1'den itibaren tekrar çalıştırmak ve yeni modeli seçmek gerekir.
- **VS Code çıktı kesiyor:** Uzun çıktılarda "View as a scrollable element" linkini kullanın.