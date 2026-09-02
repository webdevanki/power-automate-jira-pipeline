# Decyzje architektoniczne (ADR)

ADR: decyzje projektowe wraz z kontekstem, wyborem i konsekwencjami. Projekt jest wersją demo, część decyzji wynika z ograniczeń środowisk, co zostało poniżej zaznaczone.

## ADR-001: HTTP POST do REST API zamiast gotowego konektora Jira

Status. Zaakceptowane

Kontekst. Standardowy konektor "Jira Cloud, Utwórz nowy problem (v3)" w Power Automate zwracał błąd przy renderowaniu pól formularza: `Field 'reporter' of type 'user' is not supported`. Konektor nie obsługuje pól typu `user`, a obecność pola Reporter w konfiguracji projektu blokowała renderowanie wszystkich pól akcji, w tym Summary i Description. To znany, udokumentowany problem konektora.

Rozważane opcje.

1. Modyfikacja konfiguracji Jiry (usunięcie lub opcjonalne pole Reporter). Utrudnione, bo projekt korzysta z nietypowego schematu pól.
2. Bezpośrednie wywołanie REST API Jiry akcją HTTP POST, z pominięciem konektora.

Decyzja. Opcja 2: HTTP POST do `/rest/api/2/issue`.

Konsekwencje.

- Pełna kontrola nad body zgłoszenia i dowolny zestaw pól.
- Brak zależności od błędu i ograniczeń konektora.
- Wymaga ręcznego uwierzytelniania (Basic Auth + token) oraz samodzielnego zbudowania zapytania.

## ADR-002: Uwierzytelnianie tokenem API i rotacja tokenu

Status. Zaakceptowane

Kontekst. Jira Cloud przy Basic Auth wymaga tokenu API. Logowanie do Jiry odbywa się przez Google (SSO), które nie udostępnia hasła aplikacjom.

Decyzja. Basic Auth, gdzie nazwa użytkownika to e-mail konta Atlassian, a hasło to token API. Token traktowany jako sekret.

Konsekwencje.

- Integracja działa niezależnie od SSO/Google.
- Token można unieważnić bez zmiany hasła konta.
- Docelowo token powinien być przechowywany jako secure parameter lub zmienna środowiskowa, aby nie trafił do eksportu przepływu.

## ADR-003: Schemat JSON generowany z pełnej odpowiedzi API

Status. Zaakceptowane

Kontekst. Akcja "Przeanalizuj dane JSON" wymaga schematu opisującego strukturę odpowiedzi. 

Schemat wygenerowano z pełnej, rzeczywistej odpowiedzi endpointu (`/users/1`).

Konsekwencje.

- Wszystkie pola odpowiedzi dostępne jako tokeny dynamiczne.

## ADR-004: API v2 (tekst) zamiast v3 (ADF) w pierwszej iteracji

Status. Zaakceptowane

Kontekst. Jira REST API v3 wymaga pola `description` w formacie ADF (Atlassian Document Format, struktura JSON). API v2 przyjmuje opis jako zwykły tekst.

Decyzja. Pierwsza iteracja korzysta z API v2 (`/rest/api/2/issue`).

Konsekwencje.

- Prostsze, czytelne body i szybsza droga do działającego rozwiązania.
- Migracja na v3/ADF możliwa jako osobna iteracja.

## ADR-005: Dane zgłaszającego w opisie zgłoszenia, nie w polach strukturalnych Jiry

Status. Zaakceptowane

Kontekst. Dane pobierane z jsonplaceholder to dane testowe. Pola strukturalne Jiry typu `user` (np. Reporter) wymagają istniejącego konta i jego account ID, którego dla danych testowych nie ma.

Decyzja. Dane zgłaszającego (`name`, `email`) trafiają do pola `description`, złożone akcją Compose.

Konsekwencje.

- Brak zależności od kont użytkowników w instancji Jiry.
- Komplet informacji o zgłaszającym w jednym miejscu.
- W środowisku produkcyjnym rozważono by mapowanie na pole Reporter przez lookup account ID Atlassian.

## ADR-006: Status `Registered in Jira` zamiast `In Progress` po utworzeniu zgłoszenia

Status. Zaakceptowane

Kontekst. Pierwotne wymaganie (FR-06) zakładało ustawienie statusu `In Progress`. W momencie utworzenia zgłoszenia nikt jeszcze nad nim nie pracuje, więc `In Progress` byłby nieprawdziwy.

Decyzja. Przepływ ustawia status kontrolny `Registered in Jira`. Do kolumny Status dodano tę opcję.

Konsekwencje.

- Status w SharePoint odzwierciedla stan automatyzacji, nie stan pracy zespołu.
- `In Progress` pozostaje zmianą ręczną lub przyszłą synchronizacją zwrotną Jira do SharePoint.
- Rozdzielenie obu znaczeń jest świadome i opisane w mapowaniu statusów (`requirements.md`).

## ADR-007: Kontrola pola Status w formularzu tworzenia

Status. Propozycja, nie wdrożone

Kontekst. Zgłaszający może obecnie ręcznie ustawić dowolny status przy tworzeniu rekordu, co pozwala na niespójny stan początkowy.

Decyzja. Wartość domyślna `New` i ukrycie pola Status w formularzu tworzenia; status zmieniany wyłącznie przez przepływ lub obsługę procesu.

Konsekwencje.

- Spójny stan początkowy każdego zgłoszenia.
- Wymaga zmiany w formularzu SharePoint lub Power Apps, stąd wydzielone jako osobny krok.

## ADR-008: Obsługa błędów wzorcem Try/Catch

Status. Zaakceptowane

Kontekst. Power Automate nie ma natywnego try/catch. Odporność na błędy realizuje się dwoma blokami Scope i mechanizmem `Configure run after`.

Decyzja. Scope "Try" zawiera HTTP POST do Jiry oraz Update item w SharePoint. Scope "Catch" z `Configure run after` = has failed / is skipped / has timed out wysyła powiadomienie e-mail.

Konsekwencje.

- Nieudane utworzenie zgłoszenia pomija Update item, więc rekord nie dostaje fałszywego statusu.
- W treści powiadomienia w Catch tokeny muszą pochodzić z wyzwalacza (`triggerBody`), nie z akcji w Try. Akcje w Try mogły zostać pominięte, a ich tokeny są wtedy puste.
- Zweryfikowane celowym błędem (nieistniejący klucz projektu): Try failed, Update pominięty, Catch wykonany, e-mail wysłany.
