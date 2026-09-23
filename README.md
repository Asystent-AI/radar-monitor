# Czujnik radaru Głosu Czemiernik

Radar redakcji (`redaktor.fg.pl`, kontener `glos-radar`) sam pilnuje swoich
usług: modelu AI, Signala i webhooków Inoreadera. Nie jest jednak w stanie
zgłosić własnej śmierci — gdy padnie kontener albo cała maszyna, milkną razem
z nim wszystkie jego kanały alarmowe, a cisza wygląda dokładnie tak samo jak
spokój.

Na samym serwerze stoi dozór poza kontenerem (`/opt/glos-radar/doglad.sh`,
cron co 5 minut): sprawdza radar od środka, sam go podnosi i pisze na Signal,
gdy to nie pomaga. Nie zobaczy jednak sytuacji, w której padnie cała maszyna
albo ruch nie dochodzi z zewnątrz (Traefik, certyfikat, sieć operatora).

To repozytorium jest jedyną częścią systemu, która stoi poza tamtym serwerem.
Co 10 minut pyta `https://inoreader.fg.pl/radar/health` (trzy podejścia, żeby
chwilowa sieć nikogo nie budziła) i:

- gdy radar milczy — zakłada zgłoszenie z etykietą `radar-down` (powiadomienie
  e-mail z GitHuba) i wysyła push przez [ntfy](https://ntfy.sh),
- gdy radar odpowiada, ale ma odłączony Signal — to samo z etykietą `signal-down`,
- gdy wszystko wraca do normy — dopisuje komentarz, zamyka zgłoszenie i wysyła
  push z odwołaniem alarmu.

Push idzie na temat ntfy trzymany w sekrecie repozytorium `NTFY_TOPIC`.
Żeby dostawać powiadomienia na telefon: aplikacja **ntfy** (Android, iOS) →
Subscribe to topic → nazwa tematu. Bez aplikacji zostaje e-mail z GitHuba.

Repozytorium jest publiczne, bo GitHub Actions w publicznych repozytoriach nie
zużywają minut z limitu. Nie ma tu żadnych sekretów poza tematem push, a adres
`/radar/health` i tak jest jawny (zwraca tylko stan usług, bez treści).

## Co uruchamia czujnik (od 23.09.2026)

Harmonogram (`schedule`) ruszył dopiero wieczorem 22.09 i GitHub puszcza go
co 2 do 5 godzin, choć cron mówi „co 5 minut”. To znane dławienie częstych
harmonogramów na darmowych kontach, więc sam harmonogram nie nadaje się na
szybki alarm.

Dlatego głównym wyzwalaczem jest zdarzenie `push` na gałęzi `puls`. Dozór na
serwerze (`/opt/glos-radar/doglad.sh`) co 15 minut robi z `main` gałąź `puls`
z jednym pustym commitem i wypycha ją siłą kluczem wdrożeniowym tylko do tego
repozytorium (`/root/.ssh/radar-monitor`, klucz „doglad-radaru” w Settings →
Deploy keys). Zdarzeń push GitHub nie dławi, więc czujnik patrzy na radar
z zewnątrz co 15 minut. Historia `main` się nie zmienia.

Gdy puls ustaje (padł serwer albo sieć), czujnik dalej ruszy z harmonogramu,
a niezależnie od niego po 40 minutach dochodzi alarm martwego człowieka
z ntfy. Push o odłączonym Signalu wysyła dozór na serwerze, więc czujnik
zakłada w tej sprawie tylko zgłoszenie (e-mail z GitHuba), bez drugiego pusha.
