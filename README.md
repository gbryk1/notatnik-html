# Notatnik

Jednoplikowe, statyczne narzędzie HTML do przeglądania, edycji i archiwizowania
notatek przechowywanych bezpośrednio w repozytorium GitHub. Brak backendu, brak
bazy danych — repozytorium git jest jedynym źródłem prawdy.

Docelowy scenariusz: notatki (tekst, screeny, linki, luźne myśli) trafiają do
folderu `inbox/` z telefonu przez bota Telegram + automatyzację (poza zakresem
tego narzędzia). `index.html` to desktopowy etap przetwarzania: otwierasz go,
przeglądasz nowe notatki i albo je edytujesz i archiwizujesz, albo usuwasz.

Działa zarówno otwarty lokalnie (`file://`), jak i hostowany jako statyczna
strona (np. GitHub Pages) — cała komunikacja z GitHubem odbywa się przez
REST Contents API wywoływane bezpośrednio z przeglądarki (`fetch`).

## Struktura repozytorium

```
repo/
  inbox/      - nowe, nieprzejrzane notatki
  archiwum/   - notatki przejrzane / przetworzone
  index.html  - samo narzędzie (serwowane przez GitHub Pages z katalogu głównego)
```

Każdy plik to jedna notatka. Notatki tekstowe mają rozszerzenie `.md`, obrazy
`.png/.jpg/.gif/.webp`. Nazwa pliku to znacznik czasu ISO-8601 (z dwukropkami
zamienionymi na myślniki) plus slug tytułu, np.:

```
2026-09-25T14-30-00--tytul-notatki.md
```

Dzięki temu lista plików sortuje się chronologicznie, a nazwy są bezpieczne
dla systemu plików i URL-i.

## Konfiguracja

Przy pierwszym uruchomieniu `index.html` poprosi o:

- **Token dostępu GitHub (PAT)** — z uprawnieniami do odczytu/zapisu zawartości
  repozytorium.
- **Repozytorium** w formacie `owner/repo`.
- **Gałąź** (domyślnie `main`).
- **Nazwę folderu skrzynki** (domyślnie `inbox`).
- **Nazwę folderu archiwum** (domyślnie `archiwum`).

Po zapisaniu narzędzie testuje połączenie (GET do repozytorium) i zapisuje
konfigurację w `localStorage` przeglądarki. Ustawienia można później zmienić
przyciskiem ⚙ w prawym górnym rogu.

### Zalecany token

Zamiast tokenu z pełnym dostępem do konta, zalecany jest **fine-grained
Personal Access Token** ograniczony wyłącznie do tego repozytorium, z
uprawnieniami **Contents: Read and write**. Token trafia wyłącznie do
`localStorage` przeglądarki — nigdy nie jest zapisywany w repozytorium ani
wysyłany nigdzie poza `api.github.com`.

## Znane ograniczenia

- Token przechowywany jest jawnym tekstem w `localStorage` — konfiguracja
  działa tylko w jednej przeglądarce, bez synchronizacji między urządzeniami.
- Każda akcja (zapis, przeniesienie, usunięcie) to osobny commit w historii
  repozytorium — historia może rosnąć szybko.
- Brak Git LFS — tylko zwykłe pliki.
- Brak rozwiązywania konfliktów — API GitHub po prostu zwróci błąd (np. 409),
  który zostanie pokazany w interfejsie.
