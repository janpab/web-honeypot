Projekt to autorski Web Honeypot (pułapka na skanery, boty hakerskie i bug-hunterów), symulujący wyciek wewnętrznego panelu deweloperskiego firmy z sektora energetycznego (energysystems.uk).
Celem projektu było stworzenie środowiska, które w sposób zautomatyzowany zbiera, parsuje i wizualizuje złośliwy ruch sieciowy (skanowanie podatności, próby logowania, scraping) przy wykorzystaniu bardzo ograniczonych zasobów serwerowych, wspomagając się chmurą i lokalnym systemem analitycznym.
Wykorzystane technologie: Nginx (Custom JSON Logging), Cloudflare (DNS/SSL), Linux, Splunk Enterprise


Ze względu na ograniczenia hostingu (Mikrus – współdzielony adres IP, przydzielony niestandardowy port), architektura została zaprojektowana w modelu hybrydowym:
Frontend (Cloudflare): Obsługuje ruch HTTPS, maskuje prawdziwy port serwera i nadaje domenie wiarygodność (Flexible SSL).
Backend (VPS): Nginx nasłuchujący na niestandardowym porcie. Serwuje specjalnie spreparowaną stronę (pułapkę) HTML oraz plik security.txt z fałszywą ofertą Bug Bounty. Loguje każde zapytanie w formacie JSON.
Log Collector (Lokalny Host): Skrypt PowerShell pracujący w pętli, cyklicznie pobierający logi JSON przez protokół SCP (SSH) bez obciążania serwera VPS.
SIEM (Splunk): Lokalna instancja Splunk Enterprise, która na bieżąco analizuje pobrane pliki i generuje dasboardy (Top IP, GeoIP, Target URLs).


Problemy: 
Środowisko VPS nie posiadało własnego adresu IPv4, a jedynie przypisany niestandardowy port (np. 3369). Uniemożliwiało to standardową weryfikację domeny przez Let's Encrypt (Certbot zawsze wymusza ruch przez port 80), co skutkowało błędami 404 i blokadami (Rate Limits) ze strony organu certyfikującego.
Rozwiązanie: Zrezygnowałem z lokalnego certyfikatu na rzecz Cloudflare Proxy. Włączenie trybu Flexible SSL w Cloudflare pozwoliło wystawić certyfikat brzegowy (Edge Certificate) na porcie 443 dla użytkowników/botów, podczas gdy ruch do serwera docelowego był tunelowany po zwykłym HTTP na mój niestandardowy port.

Jak przyciągnąć atakujących (Problem "Zimnego Startu")
Problem: Nowa subdomena z dala od głównych skanerów była omijana przez boty. Samo postawienie serwera nie generowało ruchu.
Rozwiązanie: Zastosowałem techniki Active Baiting. Spreparowałem fałszywe zrzuty kodu i logów konfiguracyjnych sugerujące wyciek z infrastruktury krytycznej (SCADA/Industrial), które opublikowałem w serwisach takich jak Pastebin i urlscan.io. Dodatkowo wdrożyłem prowokujący plik /.well-known/security.txt z obietnicą wysokiej nagrody za znalezienie błędu, co błyskawicznie przyciągnęło zautomatyzowane skanery "bug bounty".

Rezultat
Projekt w pełni spełnił swoje zadanie. Dzięki integracji ze Splunkiem mam obecnie pełną widoczność złośliwego ruchu w czasie rzeczywistym. Widzę, jakie narzędzia (User-Agents) są używane do skanowania, z jakich krajów pochodzą najczęstsze ataki i jakich ukrytych ścieżek (np. /.env, /wp-admin, /backup.sql) najchętniej szukają dzisiejsze boty.

<h1>1. Kraje</h1>
<img width="936" height="382" alt="image" src="https://github.com/user-attachments/assets/f4b592ab-1f92-4a57-aac8-1075602c517b" />
<br>
<h1>2. URL-e</h1>
<img width="944" height="356" alt="image" src="https://github.com/user-attachments/assets/b3f7758e-413e-45cc-81a8-2ed3dbfd63b1" />

<br><br>
<h1>3. User Agents</h1>
<img width="890" height="357" alt="image" src="https://github.com/user-attachments/assets/e399a54a-2237-4655-a7ba-d8c4f3df21af" />
Większość zapytań to leniwy Brute-Force z curla. Okoła 200 zapytań z udawanych Chrome i Safari oraz 15 zapytań z ClaudeBota możliwe że pochodz z Pastebina lub rejsetrów certyfikatów.
