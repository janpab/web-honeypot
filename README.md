🍯 Industrial Web Honeypot & SIEM Integration
Autorski Web Honeypot symulujący wyciek panelu deweloperskiego firmy energetycznej (energysystems.uk). System automatycznie zbiera, parsuje i wizualizuje próby ataków, scraping oraz skanowanie podatności w czasie rzeczywistym.

🏗️ Architektura Systemu
Ze względu na ograniczenia hostingu (Mikrus – współdzielony adres IP, niestandardowy port), zastosowano model hybrydowy:

Frontend (Cloudflare): Obsługuje ruch HTTPS na porcie 443, maskuje port źródłowy VPS-a i nadaje domenie wiarygodność przez Flexible SSL.

Backend (VPS): Nginx na niestandardowym porcie serwujący "pułapkę" HTML oraz plik security.txt z fałszywą ofertą Bug Bounty. Logowanie odbywa się do formatu JSON.

Log Collector: Skrypt PowerShell (Lokalny Host) pracujący w pętli, pobierający logi przez SCP (SSH) bez obciążania zasobów VPS.

SIEM (Splunk): Lokalna instancja analizująca dane i generująca dashboardy analityczne.

🛠️ Rozwiązane Problemy
1. Wyzwania Sieciowe & SSL
Problem: Brak własnego adresu IPv4 i portu 80/443 uniemożliwiał standardową certyfikację Let's Encrypt.
Rozwiązanie: Wykorzystanie Cloudflare Proxy. Pozwoliło to na wystawienie certyfikatu brzegowego (Edge Certificate) na standardowym porcie 443, podczas gdy ruch do serwera docelowego tunelowany był po HTTP na niestandardowy port (np. 3369).

2. Problem "Zimnego Startu" (Active Baiting)
Problem: Nowa domena nie generowała ruchu botów.
Rozwiązanie: Zastosowanie technik nęcenia hakerów:

Publikacja fałszywych logów sugerujących wyciek infrastruktury SCADA/Industrial na serwisach Pastebin oraz urlscan.io.

Wdrożenie pliku /.well-known/security.txt z obietnicą nagrody ($5000), co przyciągnęło skanery grup Bug Bounty.

📊 Analiza Wyników (Dashboardy Splunk)
1. Geolokalizacja Atakujących
<img width="1872" height="763" alt="image" src="https://github.com/user-attachments/assets/20bdc3e6-e5e8-4637-8017-e383e0addde7" />
Analiza adresów IP wykazała szerokie spektrum geograficzne napastników, z przewagą zautomatyzowanych skanerów z centrów danych.
<br>
2. Analiza Targetowanych URL-i
<img width="890" height="357" alt="image" src="https://github.com/user-attachments/assets/53f8c8fc-d886-4dba-a273-a18e36efb534" />

Największym zainteresowaniem cieszyły się pliki konfiguracyjne oraz backupy bazy danych. Boty natychmiast po wejściu na stronę główną próbowały uzyskać dostęp do ukrytych ścieżek zdefiniowanych w kodzie jako "pułapki".
<br>
3. User Agents & Profilowanie Narzędzi
<img width="768" height="308" alt="image" src="https://github.com/user-attachments/assets/648076a3-c517-46fd-a005-e611d02fd446" />

Większość zapytań to agresywne skanowanie za pomocą curl. Odnotowano również:

~200 zapytań: Zaawansowane headles-browsery (Chrome/Safari), próbujące ominąć proste filtry anty-botowe.

15 zapytań: ClaudeBot (AI Anthropic) – co potwierdza, że domena została szybko zaindeksowana po publikacji w rejestrach certyfikatów.
<br>
🚀 Wnioski
Projekt udowodnił, że przy minimalnych nakładach finansowych można stworzyć skuteczne narzędzie do monitorowania Threat Intelligence. System pozwolił na zidentyfikowanie najczęstszych wektorów ataku (polowanie na pliki .env, .git/config oraz exploity WordPressa) stosowanych obecnie przez zautomatyzowane sieci botów.
