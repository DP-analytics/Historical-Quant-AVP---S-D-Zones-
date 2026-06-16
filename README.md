## Historical Quant AVP
<img width="1250" height="741" alt="image" src="https://github.com/user-attachments/assets/32e3bf1c-c480-42a0-8f89-89b80e3a89ce" />

## 1. Czym jest ten wskaźnik i jak działa?

Wskaźnik to zaawansowana wersja **Anchored Volume Profile** (Zakotwiczonego Profilu Wolumenu). Zamiast liczyć wolumen dla całego widocznego ekranu, dzieli on czas na ścisłe, zamknięte okresy (np. 1 dzień, 1 tydzień) nazywane **Kotwicami (Anchors)**. Dla każdej kotwicy buduje niezależną mapę dystrybucji wolumenu na osi ceny, a następnie odfiltrowuje rynkowy szum, zostawiając tylko najistotniejsze poziomy płynności (**High Volume Nodes - HVN**).

---

## 2. Założenia Matematyczne (Silnik Obliczeniowy)

Aby wskaźnik nadawał się do handlu algorytmicznego i analizy ilościowej, opiera się na dwóch rygorystycznych założeniach:

* **Frakcyjna Dystrybucja Wolumenu (Zamiast Punktowej):** Standardowe wskaźniki wrzucają 100% wolumenu świecy w jej środek (HL2). Ten algorytm oblicza rozpiętość świecy (Low do High), dzieli ją na dostępne koszyki cenowe (rzędy) i "rozsmarowuje" wolumen równomiernie. Eliminujemy dzięki temu błąd pustych stref płynności na świecach z długimi knotami (tzw. *liquidity voids*).
* **Aproksymacja Delty Podaż/Popyt:** Brak danych "tick by tick" w Pine Script wymusza aproksymację. Algorytm bada proporcję ceny zamknięcia (`Close`) względem całego zasięgu świeczki (`High - Low`). Jeśli świeca zamyka się w 80% swojej wysokości, algorytm przypisze 80% wolumenu tej świecy do popytu (kupno), a 20% do podaży (sprzedaż).

---

## 3. Silnik Statystyczny – Jak dobrać metodę detekcji stref (HVN)?

Wskaźnik nie rysuje stref "na oko". Posiada 3 niezależne silniki matematyczne do odcinania nieważnego wolumenu:

### A. Kwantyle / Percentyl (Domyślne i najbezpieczniejsze)
* **Jak działa:** Sortuje wszystkie koszyki cenowe pod kątem wielkości wolumenu od najmniejszego do największego. Ustawienie **85%** oznacza, że wskaźnik odrzuci 85% koszyków z najmniejszym obrotem i narysuje strefy tylko na podstawie górnych 15% (ścisłej elity płynności).
* **Kiedy używać:** Metoda uniwersalna (nieparametryczna). Jest wysoce odporna na rynkowe anomalie i potężne piki wolumenowe na otwarciach sesji. Używaj jako standardu do handlu intraday.

### B. Z-Score (Odchylenie Standardowe)
* **Jak działa:** Oblicza średnią arytmetyczną wolumenu dla całego profilu oraz odchylenie standardowe ($\sigma$). Rysuje strefę tylko tam, gdzie wolumen przekracza średnią o zadaną wartość (np. $1.2\sigma$).
* **Kiedy używać:** Na rynkach w silnej konsolidacji (*range-bound*), gdzie rozkład wolumenu przypomina rozkład normalny (dzwon Gaussa). Gorzej radzi sobie na rynkach silnie trendujących.

### C. Prominencja (Lokalne Szczyty)
* **Jak działa:** Bada strukturę koszyków. Jeśli koszyk ma większy wolumen niż ten bezpośrednio nad nim i pod nim (jest lokalnym maksimum), zostaje zaznaczony.
* **Kiedy używać:** W silnych trendach (np. na krypto lub na NASDAQ po otwarciu cash market). Pozwala wyłapać mikro-strefy wsparcia w postaci "schodków" (*Ledge*), które przy metodzie Kwantyli zostałyby zignorowane jako zbyt małe w stosunku do wolumenu z całego dnia.

---


## 4. Praktyczne zasady używania (Złote Reguły)

### 1. Interwał Wykresu < Zasięg Kotwicy
Aby profil zakotwiczony na `1D` (Dniu) wyliczył się poprawnie, musisz ustawić wykres w TradingView na interwał wewnątrzsesyjny (np. `5m`, `15m`, `1h`). Im niższy interwał wykresu, tym dokładniej (frakcyjnie) wskaźnik ułoży wolumen na osi ceny.

### 2. Rozdzielczość (Ilość Rzędów)
* Ustaw **50-70 rzędów** dla aktywów o średniej zmienności.
* Ustaw **80-100 rzędów** dla bardzo zmiennych indeksów (np. NQ, DAX), aby strefy nie były zbyt "grube" i mało precyzyjne. Zbyt duża rozdzielczość na niskiej zmienności wygeneruje szum.

### 3. Odczytywanie dPOC (Developing Point of Control)
Złota linia dPOC pokazuje, jak w czasie trwania profilu przemieszczał się "punkt największego obrotu".
* **Migracja dPOC za ceną:** Potwierdza siłę trendu (np. cena rośnie, dPOC przeskakuje wyżej).
* **Cena ucieka, dPOC zostaje:** Świadczy o pasywnej obronie poziomu lub dystrybucji/akumulacji (cena przebija szczyty, ale największy kapitał nadal obraca się na dole – ryzyko odwrócenia).

### 4. Włączanie MTF POC (Multi-Timeframe)
Służy do wyłapywania konfluencji. 
* **Przykład:** Kotwica profilu głównego to `1D`, a Drugi POC ustawiasz na `1W` (tygodniowy). Szukasz na wykresie momentów, gdzie jednodniowa strefa Podaży (Czerwona) nakłada się na fuksjową linię POC z całego tygodnia. To najsilniejsze geometrycznie miejsca do zajmowania pozycji *mean reversion* (powrotu do średniej).
