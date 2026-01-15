# Ataki XSS

## Zadanie 1: XSS Reflected

### Cel zadania
Zrozumienie mechanizmu **Reflected XSS**, w którym złośliwy skrypt jest przesyłany do serwera w żądaniu HTTP (np. jako parametr URL), a następnie natychmiast „odbijany” i wykonywany w przeglądarce użytkownika.



### Instrukcja krok po kroku

1.  **Lokalizacja:** W menu bocznym DVWA wybierz zakładkę **XSS (Reflected)**.
2.  **Identyfikacja pola:** Na środku strony zobaczysz formularz z pytaniem: *"What's your name?"*.
3.  **Wstrzyknięcie (Injection):** Zamiast zwykłego imienia, wprowadź w polu tekstowym poniższy kod JavaScript:
    ```html
    <script>alert('Hacked by [Twoje Imię]')</script>
    ```
4.  **Uruchomienie:** Kliknij przycisk **Submit**.

---

### Co się dzieje?
Serwer przyjmuje Twoje dane i bez żadnej weryfikacji wstawia je bezpośrednio do kodu HTML strony zwrotnej. Przeglądarka, odbierając odpowiedź od serwera, interpretuje tag `<script>` jako instrukcję do wykonania, a nie jako zwykły tekst.


> [!TIP]
> **Podpowiedź 1: Spójrz na pasek adresu**
> Po kliknięciu Submit, spójrz na URL w przeglądarce. Zobaczysz tam fragment: `?name=<script>...`. To jest dowód na to, że skrypt "podróżuje" w linku. Gdybyś wysłał ten konkretny link innej zalogowanej osobie, skrypt uruchomiłby się na jej komputerze.

---

### Pytania do refleksji 

* **Dlaczego ten atak nazywa się "odbitym"?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Ponieważ skrypt nie zostaje zapisany na stałe na serwerze, lecz jest „odbijany” od aplikacji i wraca do użytkownika w tej samej sesji (w odpowiedzi HTTP).
  </details>

* **Jak można się przed tym bronić?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Główną metodą jest tzw. **Context-Aware Output Encoding**, czyli zamiana znaków specjalnych (takich jak `<` na `&lt;` oraz `>` na `&gt;`) przed wyświetleniem ich na stronie. Dzięki temu przeglądarka wyświetli kod jako tekst, zamiast go uruchamiać.
  </details>


## Zadanie 2: XSS Stored

### Cel zadania
Zrozumienie mechanizmu **Stored XSS**, który polega na trwałym umieszczeniu złośliwego kodu w bazie danych aplikacji. Skrypt ten jest następnie automatycznie serwowany każdemu użytkownikowi, który odwiedzi zainfekowaną podstronę.


### Instrukcja krok po kroku

1.  **Lokalizacja:** W menu bocznym DVWA wybierz zakładkę **XSS (Stored)**.
2.  **Ominięcie limitu znaków (Ważne!):**
    * Pole *Message* ma domyślny limit (atrybut `maxlength`), który może uciąć Twój kod.
    * Kliknij prawym przyciskiem myszy na pole tekstowe **Message** i wybierz **Zbadaj** (Inspect).
    * W kodzie HTML odszukaj fragment `maxlength="10"` i zmień go na `maxlength="100"`.
3.  **Wypełnienie formularza:**
    * **Name:** Wpisz dowolną nazwę, np. `Admin`.
    * **Message:** Wklej poniższy payload:
    ```html
    <script>document.body.style.backgroundColor = "red";</script>
    ```
4.  **Uruchomienie:** Kliknij przycisk **Sign Guestbook**.
---

### Co się dzieje? 
W przeciwieństwie do ataku Reflected, Twój skrypt nie „odbił się” tylko raz. Został on wysłany do serwera i zapisany w bazie danych jako nowa wiadomość w księdze gości. Teraz, za każdym razem, gdy przeglądarka (Twoja lub innego użytkownika) pobiera listę wiadomości, pobiera również Twój skrypt i natychmiast go wykonuje.


> [!TIP]
> **Podpowiedź 1: Testowanie trwałości**
> Przejdź do innej zakładki (np. Brute Force), a następnie wróć do **XSS (Stored)**. Zauważ, że strona automatycznie robi się czerwona bez wpisywania czegokolwiek. 

> [!TIP]
> **Podpowiedź 2: Dlaczego nic nie widzę w wiadomościach?**
> Jeśli na liście komentarzy widzisz puste pole obok swojego imienia, to dobry znak! Oznacza to, że przeglądarka zinterpretowała Twój tekst jako kod i go „ukryła”, wykonując instrukcję zmiany koloru. Sprawdź kod źródłowy strony (**Ctrl + U**), aby zobaczyć swój skrypt siedzący wewnątrz tabeli HTML.

> [!TIP]
> **Podpowiedź 3: Sprzątanie po ataku**
> Aby „naprawić” stronę i usunąć czerwone tło, musisz wyczyścić bazę danych. Możesz to zrobić przyciskiem **Clear Guestbook** pod formularzem lub poprzez **Setup / Reset DB** w menu głównym.

---

### Pytania do refleksji 

* **Gdzie fizycznie znajduje się kod ataku w przypadku Stored XSS?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Kod znajduje się w bazie danych serwera (np. w tabeli przechowującej komentarze lub wpisy księgi gości).
  </details>

* **Dlaczego ten typ ataku jest uważany za groźniejszy niż Reflected?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Ponieważ nie wymaga wysyłania ofierze podejrzanych linków. Ofiara zostaje zaatakowana automatycznie, po prostu wchodząc na legalną i zaufaną podstronę serwisu, która została wcześniej zainfekowana.
  </details>

---

## Zadanie 3: XSS DOM-Based – Manipulacja po stronie klienta

### Cel zadania
Zrozumienie mechanizmu **DOM XSS**, w którym podatność znajduje się w skryptach JavaScript na stronie (po stronie klienta). Atak polega na modyfikacji środowiska **DOM**  w taki sposób, aby przeglądarka wykonała nieoczekiwany kod.



### Instrukcja krok po kroku

1.  **Lokalizacja:** W menu bocznym DVWA wybierz zakładkę **XSS (DOM)**.
2.  **Obserwacja:** Na stronie zobaczysz listę rozwijaną z wyborem języka. Wybierz dowolny język i kliknij **Select**.
3.  **Analiza URL:** Spójrz na pasek adresu. Zobaczysz parametr: `?default=English`. Skrypt JavaScript na tej stronie pobiera tę wartość i wypisuje ją bezpośrednio do kodu strony.
4.  **Wstrzyknięcie (Injection):** Zmodyfikuj ręcznie pasek adresu URL, dopisując po znaku `=` swój kod. Cały adres powinien wyglądać następująco:
    ```text
    http://localhost:8080/vulnerabilities/xss_d/?default=English<script>alert(document.cookie)</script>
    ```
5.  **Uruchomienie:** Naciśnij **Enter**, aby odświeżyć stronę z nowym adresem URL.

---

### Co się dzieje? 
W tym przypadku serwer często w ogóle nie bierze udziału w ataku. To skrypt JavaScript już obecny na stronie (wykorzystujący np. obiekt `document.location`) pobiera fragment tekstu z adresu URL i „wstrzykuje” go do drzewa DOM strony. Przeglądarka widzi nowy element `<script>` i natychmiast go wykonuje, wyświetlając Twoje ciasteczka sesyjne.

---


> [!TIP]
> **Podpowiedź 1: Gdzie jest błąd w kodzie?**
> Naciśnij **Ctrl + U** . Zauważysz tam skrypt, który używa `indexOf("default=")` i `document.write`. To właśnie `document.write` jest tutaj „dziurą”, bo bezkrytycznie wpisuje do dokumentu wszystko, co znajdzie w adresie URL.

> [!TIP]
> **Podpowiedź 2: Kradzież ciasteczek (Cookies)**
> Użycie `alert(document.cookie)` to klasyczny sposób na sprawdzenie, czy sesja użytkownika jest bezpieczna. W prawdziwym ataku haker wysłałby te dane na swój serwer, co pozwoliłoby mu przejąć sesję użytkownika bez znajomości hasła.

> [!TIP]
> **Podpowiedź 3: Spróbuj bez odświeżania**
> DOM XSS często wykorzystuje znak `#` (fragment/kotwicę) zamiast `?`. Spróbuj zamienić w adresie URL `?default=` na `#default=` i dopisać skrypt. Zauważ, że w wielu przypadkach dane po `#` nie są nawet wysyłane do serwera, ale JavaScript na stronie nadal może je przetworzyć i wykonać atak.

---

### Pytania do refleksji

* **Dlaczego DOM XSS jest trudny do wykrycia przez systemy Firewall (WAF)?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Ponieważ manipulacja odbywa się w przeglądarce. Jeśli haker użyje znaku `#`, zawartość po nim nie jest przesyłana do serwera w żądaniu HTTP, więc systemy ochrony sieciowej (WAF) nie mają szansy przeanalizować złośliwego ładunku.
  </details>

* **Jak programista może naprawić ten błąd?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Zamiast używać niebezpiecznych metod jak `document.write()`, należy stosować bezpieczniejsze alternatywy, takie jak `.textContent` lub `.innerText`. Te metody traktują dane wyłącznie jako zwykły tekst i nie interpretują ich jako kodu HTML czy JavaScript.
  </details>

---


## Zadanie 4: Omijanie filtrów

### Cel zadania
Nauczenie się omijania prostych mechanizmów obronnych opartych na „czarnych listach” (*blacklists*). Dowiesz się, dlaczego usuwanie konkretnych słów kluczowych (np. `<script>`) nie jest skutecznym zabezpieczeniem.

### Konfiguracja startowa
1. Przejdź do zakładki **DVWA Security**.
2. Ustaw **Security Level** na **Medium** i kliknij **Submit**.
3. Wróć do zakładki **XSS (Reflected)**.

---

### Instrukcja krok po kroku

1.  **Próba podstawowa:** Spróbuj wpisać payload z Zadania 1: 
    ```html
    <script>alert(1)</script>
    ```
2.  **Obserwacja:** Zauważysz, że skrypt nie działa, a na stronie wyświetla się tylko `alert(1)`. Tag `<script>` został wycięty przez serwer.
3.  **Analiza (View Source):** Kliknij przycisk **View Source** na dole strony. Zobaczysz, że programista użył funkcji `str_replace('<script>', '', ... )`.
4.  **Ominięcie nr 1 (Wielkość liter):** Funkcja `str_replace` w PHP na tym poziomie jest czuła na wielkość liter. Wpisz:
    ```html
    <sCrIpT>alert('Ominięte!')</sCrIpT>
    ```
5.  **Ominięcie nr 2:** Możesz wywołać JavaScript bez użycia tagu `<script>`, korzystając np. z atrybutów zdarzeń w innych tagach. Wpisz:
    ```html
    <img src=x onerror=alert('Hacked_przez_img')>
    ```

---

### Co się dzieje?
Programista założył, że haker zawsze użyje małych liter w tagu `<script>`. To klasyczny błąd „czarnej listy”. Atakujący może użyć alternatywnych tagów (jak `<img>`, `<body>`, `<iframe>`) lub zdarzeń (*Events*) takich jak `onerror`, `onload`, `onmouseover`, których prymitywny filtr nie bierze pod uwagę.

---


> [!TIP]
> **Podpowiedź 1: Sprytne zagnieżdżanie**
> Czasami filtry usuwają frazę `<script>` tylko raz i nie robią tego rekurencyjnie. Co się stanie, jeśli wpiszesz `<scr<script>ipt>`? Jeśli filtr usunie środkowy tag, to zewnętrzne litery połączą się, tworząc ponownie działający tag `<script>`.

> [!TIP]
> **Podpowiedź 2: Event Handlers**
> Atrybut `onerror` w tagu `<img>` wykonuje kod JavaScript w momencie, gdy obrazek nie może zostać wczytany (a ponieważ daliśmy `src=x`, to na pewno się nie wczyta). To obecnie jedna z najpopularniejszych metod omijania filtrów XSS, ponieważ tag `<img>` jest zazwyczaj dozwolony w wielu systemach.

---

### Pytania do refleksji

* **Dlaczego zamiana `<script>` na pusty ciąg znaków jest niebezpieczna?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Ponieważ istnieją dziesiątki innych tagów HTML (np. `<img>`, `<a>`, `<iframe>`, `<svg>`), które mogą uruchomić JavaScript poprzez atrybuty zdarzeń. Dodatkowo, proste filtry tekstowe można ominąć zmieniając wielkość liter lub stosując techniki zagnieżdżania tagów.
  </details>

* **Jak można poprawić ten filtr, aby był bardziej odporny?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Należy używać funkcji obsługujących wyrażenia regularne (np. `preg_replace`) z flagą ignorującą wielkość liter. Jednak najlepszym rozwiązaniem nie jest usuwanie znaków (czarna lista), lecz **kodowanie danych wyjściowych** (output encoding) – zamiana wszystkich znaków `<` i `>` na ich encje HTML.
  </details>

---

## Zadanie 5: Zaawansowane omijanie filtrów – Poziom High

### Cel zadania
Nauczenie się omijania restrykcyjnych filtrów, które blokują standardowe tagi i atrybuty zdarzeń. Zrozumiesz, że dopóki aplikacja pozwala na wstrzykiwanie dowolnych tagów HTML, zawsze istnieje ryzyko wykonania kodu JavaScript.

### Konfiguracja startowa
1. Przejdź do zakładki **DVWA Security**.
2. Ustaw **Security Level** na **High** i kliknij **Submit**.
3. Wróć do zakładki **XSS (Reflected)**.

---

### Instrukcja krok po kroku

1.  **Weryfikacja zabezpieczeń:**
    Spróbuj użyć payloadów z poprzednich zadań:
    * `<sCrIpT>alert(1)</sCrIpT>`
    * `<img src=x onerror=alert(1)>`
    
    **Obserwacja:** Żaden z nich nie działa. Filtr na poziomie High jest znacznie inteligentniejszy i wycina większość znanych fraz związanych ze skryptami.

2.  **Analiza kodu:**
    Spójrz w kod źródłowy PHP. Zobaczysz użycie wyrażeń regularnych (`preg_replace`), które szukają frazy `script` w sposób ignorujący wielkość liter. Blokowane są też popularne atrybuty zdarzeń.

3.  **Wykorzystanie tagu SVG (Scalable Vector Graphics):**
    Tag `<svg>` jest częścią standardu HTML5 i posiada własne mechanizmy uruchamiania skryptów, które często są pomijane przez filtry skupione na tagach `<img>` czy `<iframe>`. 
    Wprowadź payload:
    ```html
    <svg onload="alert('XSS na poziomie High')">
    ```

4.  **Alternatywa: Tag `<a>` z protokołem `javascript:`:**
    Jeśli filtr blokuje zdarzenia typu `onload`, można spróbować zmusić użytkownika do interakcji. Wpisz:
    ```html
    <a href="javascript:alert('XSS')">Kliknij tutaj po prezent!</a>
    ```

---

### Co się dzieje?
Na poziomie High programista próbował zablokować wszystko, co kojarzy mu się z „aktywnym” kodem. Jednak standard HTML jest tak obszerny, że niemal niemożliwe jest zablokowanie wszystkich kombinacji.

* **Tag SVG:** Jest interpretowany przez przeglądarkę jako obraz wektorowy, ale posiada własne zdarzenie `onload`, które wykonuje się natychmiast po wyrenderowaniu grafiki.
* **Pseudo-protokoły:** Atrybut `href` w linkach może zamiast adresu URL zawierać bezpośrednie polecenie JavaScript (`javascript:`), które zostanie wykonane po kliknięciu.

---

> [!TIP]
> **Podpowiedź 1: Dlaczego SVG działa?**
> Filtry często są budowane „pod konkretne zagrożenia” (np. `<script>`). Programiści rzadko pamiętają, że elementy grafiki wektorowej SVG również mogą przenosić kod JavaScript. To świetny przykład ataku typu *Out-of-the-box*.

> [!TIP]
> **Podpowiedź 2: Obfuscation (Zaciemnianie)**
> Jeśli filtr blokuje konkretne znaki, hakerzy używają kodowania (np. kodów ASCII lub HEX). Zamiast pisać `alert`, można zapisać to jako `\u0061lert`. Przeglądarka to zrozumie, ale prosty filtr tekstowy na serwerze – nie.

---

### Pytania do refleksji

* **Czy czarna lista (blacklist) może kiedykolwiek być w 100% skuteczna?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  Nie, ponieważ standardy internetowe (HTML/JS) ciągle ewoluują, a liczba kombinacji tagów, atrybutów i metod kodowania znaków jest zbyt duża, by przewidzieć wszystkie możliwe wektory ataku.
  </details>

* **Jaka jest różnica między `str_replace` (Medium) a `preg_replace` (High)?**
  <details>
  <summary>Kliknij, aby zobaczyć odpowiedź</summary>
  `str_replace` szuka dosłownego, konkretnego ciągu znaków. `preg_replace` pozwala na użycie wyrażeń regularnych (Regex), co umożliwia wyszukiwanie wzorców (np. dowolnej wielkości liter, dodatkowych spacji czy znaków specjalnych wewnątrz tagu).
  </details>

---
