# Biblioteka do generowania wizualizacji PDF faktur i UPO

Biblioteka do generowania wizualizacji PDF faktur oraz UPO na podstawie plików XML.  
Obsługuje zarówno **aplikację webową (frontend)**, jak i **serwer Node.js API (backend)**.

---
## 📦 Architektura projektu

Projekt składa się z **trzech głównych komponentów**:

### 1. **Biblioteka frontendowa** (`lib-public/`)
- Generowanie PDF po stronie klienta (przeglądarka)
- Eksportowana jako pakiet npm
- Używa **pdfMake** i **fast-xml-parser**

### 2. **Aplikacja demonstracyjna** (`src/`)
- Interfejs webowy do testowania biblioteki
- Uruchamiany przez Vite (dev server)
- Dostępny pod adresem `http://localhost:5173`

### 3. **⚙️ Serwer Node.js API** (`src/server/`)
- **REST API** do generowania PDF-ów po stronie serwera
- Używa tej samej biblioteki co frontend (dzięki **browser polyfills**)
- Port: `3000` (domyślnie)
- Technologie: **Express.js**, **TypeScript**, **Docker**

---
## 📦 0. Instalacja zależności 
### 0.1 Wymagania wstępne 
Przed rozpoczęciem upewnij się, że masz zainstalowane:
- **Node.js** w wersji **22.14.0** lub nowszej. Sprawdź wersję: `node --version`  Pobierz z: [https://nodejs.org](https://nodejs.org)
- **npm** (instalowany automatycznie z Node.js).  Sprawdź wersję: `npm --version`  Zalecana: **10.x** lub nowsza 
- **Docker** (opcjonalnie, dla konteneryzacji). Pobierz z: [https://www.docker.com/](https://www.docker.com/) ### 0.2 Sklonuj repozytorium 

 ```bash 
 git clone https://github.com/LeMobi/ksef-pdf-generator 
 cd ksef-pdf-generator
 ```

## 0.3 Instalacja zależności projektu

Po sklonowaniu repozytorium, zainstaluj wszystkie wymagane pakiety:

```bash
# Instaluj zależności 
npm install
```

**Alternatywnie** (zalecane dla CI/CD i produkcji):

```bash
# npm ci instaluje dokładnie wersje z package-lock.json
npm ci
```
## 0.4 Różnica między `npm install` i `npm ci`

|Komenda|Kiedy używać|Opis|
|---|---|---|
|`npm install`|Rozwój lokalny|Instaluje/aktualizuje zależności, może zmieniać `package-lock.json`|
|`npm ci`|CI/CD, produkcja|Instaluje dokładne wersje z `package-lock.json`, szybsze, deterministyczne|

## 0.5 Weryfikacja instalacji

Po instalacji sprawdź, czy wszystko działa:
```bash
# Sprawdź zainstalowane pakiety 
npm list --depth=0 

# Sprawdź czy nie ma problemów 
npm audit 

# Opcjonalnie: napraw podatności 
npm audit fix
```


**Oczekiwany output:**

```text
ksef-pdf-generator@0.0.42 
├── @kmf/ksef-fe-invoice-converter@0.0.42 
├── express@5.0.1 
├── typescript@5.7.3 
└── ... (pozostałe pakiety)`
```

## 0.6 Rozwiązywanie problemów instalacji

## Problem: `npm install` zwraca błędy

**Rozwiązanie 1: Wyczyść cache i node_modules**

```bash
# Usuń node_modules i package-lock.json 
rm -rf node_modules package-lock.json 

# Wyczyść cache npm 
npm cache clean --force 

# Zainstaluj ponownie 
npm install
```

**Rozwiązanie 2: Użyj innej wersji Node.js**

```bash
# Zainstaluj nvm (Node Version Manager) 
# Windows: https://github.com/coreybutler/nvm-windows 
# Linux/Mac: https://github.com/nvm-sh/nvm 

# Zainstaluj Node.js 22 
nvm install 22 
nvm use 22 

# Zainstaluj zależności 
npm install
```

**Rozwiązanie 3: Sprawdź uprawnienia (Linux/Mac)**

```bash
# Jeśli błędy związane z uprawnieniami 
sudo chown -R $(whoami) ~/.npm 
sudo chown -R $(whoami) node_modules`
```

---

## 🔧 Dostępne komendy npm

Projekt zawiera następujące skrypty npm (zdefiniowane w `package.json`):

## Buildowanie

```bash
 npm run build:lib       # Zbuduj bibliotekę (frontend) 
 npm run build:server # Zbuduj serwer Node.js 
 npm run build    # Zbuduj bibliotekę + serwer`
```

## Produkcyjne

```bash
npm start           # Uruchom serwer produkcyjny (wymaga wcześniejszego buildu)
```
## Testowanie

```bash
npm test                 # Uruchom testy 
npm run test:ui          # Uruchom testy z interfejsem graficznym 
npm run test:ci          # Uruchom testy z raportem coverage`
```
## Czyszczenie

```bash
# Wyczyść projekt 
rm -rf node_modules dist dist-server npm install
```
---
## 🖥️ 1. Serwer Node.js API

Serwer Node.js pozwala generować PDF-y faktur i UPO poprzez REST API.
Używa **tej samej biblioteki** co frontend, dzięki systemowi **browser polyfills**.

### 1.1 Jak to działa?

Biblioteka KSeF została pierwotnie stworzona dla przeglądarek i wykorzystuje Browser API:

- `Blob`, `File`, `FileReader`
- `document.createElement`
- `URL.createObjectURL`

**Rozwiązanie:** Plik `src/server/browser-polyfills.ts` emuluje te API w Node.js, dzięki czemu:
✅ Nie trzeba przepisywać kodu biblioteki
✅ Frontend i backend używają **tej samej logiki**
✅ Łatwa aktualizacja biblioteki bez zmian w backendzie

### 1.2 Struktura serwera

```
ksef-pdf-generator/
├── src/
│   └── server/              # Node.js API Server│       
│       ├── pdfService.ts     # Serwis generujący PDF (wrapper dla biblioteki)
│       └── browser-polyfills.ts  # Emulacja Browser API dla Node.js
└── server.ts        # Express server (REST API)
```

### 1.3 Uruchomienie serwera (lokalnie)

```bash
# Build serwera
npm run build:server

# Start serwera
npm start
```

Serwer uruchomi się pod adresem: **http://localhost:3000**

### 2.4 REST API Endpoints

#### **POST `/invoice`** - Generowanie PDF faktury

**Request:**

```bash
curl -X POST http://localhost:3000/invoice \
  -H "Content-Type: text/xml" \
  -H "X-KSeF-Number: 1234567890-20260121-ABC123XYZ-12" \
  -H "X-KSeF-QRCode: https://qr-test.ksef.mf.gov.pl/invoice/..." \
  --data @examples/invoice.xml \
  -o invoice.pdf
```

**Headers (opcjonalne):**

- `X-KSeF-Number` - Numer referencyjny KSeF (fallback: `"Numer faktury nie został przydzielony"`)
- `X-KSeF-QRCode` - Link do QR code (fallback: pusty string)

**Response:**

- **200 OK** - PDF (application/pdf)
- **400 Bad Request** - Pusty XML
- **500 Internal Server Error** - Błąd generowania PDF

---

#### **POST `/upo`** - Generowanie PDF UPO

**Request:**

```bash
curl -X POST http://localhost:3000/upo \
  -H "Content-Type: text/xml" \
  --data @examples/upo.xml \
  -o upo.pdf
```

**Response:**

- **200 OK** - PDF (application/pdf)
- **400 Bad Request** - Pusty XML
- **500 Internal Server Error** - Błąd generowania PDF

---

#### **GET `/health`** - Health check

```bash
curl http://localhost:3000/health
```

**Response:**

```json
{
  "status": "ok",
  "timestamp": "2026-01-21T10:30:15.123Z"
}
```


---

## 🐳 3. Docker

Serwer można uruchomić w kontenerze Docker.

### 3.1 Budowanie obrazu Docker

```bash
docker build -t ksef-pdf-generator .
```


### 3.2 Uruchomienie kontenera

```bash
docker run -d -p 3000:3000 --name ksef-server ksef-pdf-generator
```


### 3.3 Testowanie kontenera

```bash
# Health check
curl http://localhost:3000/health

# Generowanie PDF
curl -X POST http://localhost:3000/invoice \
  -H "Content-Type: text/xml" \
  --data @examples/invoice.xml \
  -o invoice.pdf
```

### 3.4 🚀 **Eksport obrazu do pliku (jeśli chcesz go przenieść)**

Jeśli chcesz zapisać obraz jako plik `.tar` (np. do przeniesienia na inny serwer):

```bash
# Zapisz obraz do pliku 
docker save ksef-pdf-generator:latest -o ksef-pdf-generator.tar 
# Lub skompresowany
docker save ksef-pdf-generator:latest | gzip > ksef-pdf-generator.tar.gz`

# Przywrócenie obrazu na innej maszynie
docker load -i ksef-pdf-generator.tar

# Uruchom
docker run -d -p 3000:3000 --name ksef-server ksef-pdf-generator
```

### 3.5 Logi i debugowanie

```bash
# Wyświetl logi kontenera
docker logs ksef-server -f

# Wejdź do kontenera
docker exec -it ksef-server /bin/sh

# Zatrzymaj i usuń kontener
docker stop ksef-server
docker rm ksef-server
```


---

## 📄 4. Jak wygenerować fakturę

### 4.1 Przez API (backend)

```bash
curl -X POST http://localhost:3000/invoice \
  -H "Content-Type: text/xml" \
  -H "X-KSeF-Number: 5555555555-20250808-9231003CA67B-BE" \
  --data @examples/invoice.xml \
  -o invoice.pdf
```


---

## 📄 5. Jak wygenerować UPO

### 5.1 Przez API (backend)

```bash
curl -X POST http://localhost:3000/upo \
  -H "Content-Type: text/xml" \
  --data @examples/upo.xml \
  -o upo.pdf
```


---

## 🧪 6. Testy jednostkowe

Projekt wykorzystuje **Vite** i **Vitest** jako framework testowy.

### Uruchamianie testów

```bash
# Uruchom wszystkie testy
npm run test

# Testy z interfejsem graficznym
npm run test:ui

# Testy w trybie CI z raportem pokrycia
npm run test:ci
```

**Raport pokrycia:** `/coverage/index.html`

---

## 🛠️ 7. Struktura projektu

```
ksef-pdf-generator/
├── src/
│   ├── components/          # Komponenty React (frontend)
│   ├── lib-public/          # Biblioteka generująca PDF
│   │   ├── services/        # Logika generowania PDF
│   │   ├── types/           # TypeScript typy (FA, UPO)
│   │   └── index.ts         # Public API
│   └── server/              # Node.js API Server│       
│       ├── pdfService.ts    # Wrapper dla biblioteki
│       └── browser-polyfills.ts  # Emulacja Browser API
├── examples/
│   ├── invoice.xml          # Przykładowa faktura
│   └── upo.xml              # Przykładowe UPO
├── server.ts        # Express REST API
├── Dockerfile               # Docker image definition
├── package.json
├── tsconfig.json
├── tsconfig.server.json
└── README.md
```


---

## 📚 8. Konwencje i dobre praktyki

### 8.1 Nazewnictwo zmiennych i metod

- **Polsko-angielskie nazwy** wynikają ze struktury schematu XML faktury KSeF
- Zapewnia spójność z oficjalną definicją danych


### 8.2 Struktura danych

- Interfejsy TypeScript odzwierciedlają strukturę XML
- Zachowuje hierarchię i logiczne powiązania


### 8.3 Typy i interfejsy

- Typy definiowane w `types/` oraz plikach `*.types.ts`
- Wspierają generowanie PDF i walidację danych

---

## 🔗 9. Dokumentacja narzędzi

- **Vitest** - https://vitest.dev/guide/
- **Vite** - https://vitejs.dev/guide/
- **TypeScript** - https://www.typescriptlang.org/docs/
- **Express.js** - https://expressjs.com/
- **Docker** - https://docs.docker.com/
- **pdfMake** - https://pdfmake.github.io/docs/

---

## ⚠️ 10. Uwagi

- Pliki XML muszą być zgodne z odpowiednią schemą (FA lub UPO)
- W przypadku problemów z Node.js, użyj **nvm**: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
- **Browser polyfills** umożliwiają uruchomienie biblioteki frontendowej w Node.js
- Domyślny fallback dla `nrKSeF`: `"Numer faktury nie został przydzielony"`
---
## 📧 Kontakt i wsparcie

W razie pytań lub problemów, otwórz **Issue** na GitHubie:
https://github.com/CIRFMF/ksef-pdf-generator/issues

---

## 📝 Licencja

[Podaj licencję projektu]


