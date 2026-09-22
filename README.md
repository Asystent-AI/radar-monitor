# Czujnik radaru Głosu Czemiernik

Radar redakcji (`redaktor.fg.pl`, kontener `glos-radar`) sam pilnuje swoich
usług: modelu AI, Signala i webhooków Inoreadera. Nie jest jednak w stanie
zgłosić własnej śmierci — gdy padnie kontener albo cała maszyna, milkną razem
z nim wszystkie jego kanały alarmowe, a cisza wygląda dokładnie tak samo jak
spokój.

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
