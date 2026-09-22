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

## Stan na 22.09.2026: harmonogram nie startuje

Uruchamiany ręcznie (`Run workflow` albo `gh workflow run`) czujnik działa
bezbłędnie: sprawdzono na żywo, że zatrzymany radar daje zgłoszenie i push,
a podniesiony zamyka zgłoszenie i wysyła odwołanie. Natomiast **ani jedno
uruchomienie z harmonogramu (`schedule`) dotąd nie ruszyło**, mimo że:

- workflow leży na domyślnej gałęzi i jest `active`,
- Actions są włączone, repozytorium publiczne i nowe,
- adres e-mail konta jest zweryfikowany,
- składnia crona była próbowana w trzech wariantach (`*/10`, lista minut, `*/5`),
- plik przerejestrowano pod nową nazwą, a commit przypisano do konta
  (wcześniejsze szły jako `web-flow`),
- GitHub Status nie zgłaszał incydentu.

Dopóki to się nie zmieni, jedyną działającą warstwą jest dozór na serwerze
(`/opt/glos-radar/doglad.sh`), który obsługuje wszystko poza padnięciem całej
maszyny. Domknięcie tej ostatniej luki wymaga zewnętrznej usługi monitorującej
(np. darmowy UptimeRobot na `https://inoreader.fg.pl/radar/health`).
