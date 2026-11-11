# Robyn MMM örnek çalışma

Meta Robyn (Python, robynpy) ile örnek veri seti üzerinde MMM eğitimi.

## Veri

Veri dosyaları repoda tutulmuyor. İki dosyayı indirip proje klasörüne kaydedin:

- [dt_simulated_weekly.csv](https://raw.githubusercontent.com/facebookexperimental/Robyn/main/python/src/robyn/tutorials/resources/dt_simulated_weekly.csv): haftalık model verisi
- [dt_prophet_holidays.csv](https://raw.githubusercontent.com/facebookexperimental/Robyn/main/python/src/robyn/tutorials/resources/dt_prophet_holidays.csv): ülke bazlı tatil tablosu

## Kurulum (Windows)

1. R'ı ve glmnet paketini kurun. robynpy regresyon adımında R'daki glmnet'i kullanıyor.
2. Sanal ortamı oluşturup paketleri yükleyin:

```powershell
python -m venv robyn-env
robyn-env\Scripts\Activate.ps1
$env:R_HOME = "R kurulum klasörü"
$env:RPY2_CFFI_MODE = "ABI"
pip install -r requirements.txt
```

3. Notebook'un ilk hücresindeki R_HOME yolunu kendi R kurulumunuza göre güncelleyin.