# Kozzak — zasady bezpieczeństwa strony

Strona jest statyczna: brak backendu, bazy, logowania i formularzy. Ochrona sprowadza się do
kontroli tego, skąd przeglądarka może cokolwiek załadować.

## Allowlista źródeł (CSP w `<meta http-equiv="Content-Security-Policy">` w `index.html`)
- `default-src 'self'` — domyślnie wyłącznie pliki lokalne; absolutne i obce adresy są odrzucane.
- Jawne wyjątki: `fonts.googleapis.com` (CSS czcionek), `fonts.gstatic.com` (pliki czcionek),
  `maps.google.com` / `www.google.com` (wyłącznie jako ramka z mapą).
- `img-src 'self' data:` — zdjęcia tylko z `assets/`, `data:` dla wbudowanych tekstur SVG i favicony.
- `object-src 'none'`, `base-uri 'self'`, `form-action 'none'`. Bez `upgrade-insecure-requests` — pod adresem http:// w sieci lokalnej blokowałoby to zdjęcia; HTTPS zapewnia hosting.
- Skrypt inline jest **przypięty hashem SHA-256**. Po każdej zmianie w `<script>` przelicz hash:

```bash
node -e "const fs=require('fs');let s=fs.readFileSync('index.html','utf8');const h=require('crypto').createHash('sha256').update(s.match(/<script>([\s\S]*?)<\/script>/)[1],'utf8').digest('base64');s=s.replace(/script-src 'sha256-[^']*'/,\"script-src 'sha256-\"+h+\"'\");fs.writeFileSync('index.html',s);console.log('sha256-'+h)"
```

(uruchamiać w folderze `kozzak/`). Bez tego przeglądarka odmówi uruchomienia JS — koło, status
godzin, galeria i ściana zdjęć przestaną działać, a w konsoli pojawi się „Refused to execute inline script".

## Ładowanie dynamiczne w JS
Jedyne miejsce, które dokłada zasoby w locie, to ściana zdjęć (`#photowall`). Ścieżki przechodzą
przez `localAsset()`: dopuszczalne wyłącznie `assets/<nazwa>.(jpg|jpeg|webp|png|avif)`, bez `..`,
bez `/` na początku, bez `//`, bez schematów (`http:`, `data:`, `javascript:`). Wszystko inne jest
pomijane, zanim trafi do DOM.

## Linki wychodzące
Odnośniki do Google Maps, Instagrama i Facebooka to zwykłe `<a>` z `rel="noopener"` — CSP nie
ogranicza nawigacji użytkownika po kliknięciu, tylko to, co strona sama ładuje.

## Nagłówki hostingu
`.htaccess` ustawia `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`,
`Permissions-Policy`, `Cross-Origin-Opener-Policy`, wyłącza listowanie katalogów i dostęp do tego pliku.
