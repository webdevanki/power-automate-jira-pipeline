# Decyzje architektoniczne (ADR)

ADR: decyzje projektowye wraz z kontekstem, wyborem i konsekwencjami. Projekt jest wersją demo, część decyzji wynika z ograniczeń środowisk, co zostało poniżej zaznaczone.

## ADR-001: HTTP POST do REST API zamiast gotowego konektora Jira

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

Kontekst. Jira Cloud przy Basic Auth wymaga tokenu API. Logowanie do Jiry odbywa się przez Google (SSO), które nie udostępnia hasła aplikacjom.

Decyzja. Basic Auth, gdzie nazwa użytkownika to e-mail konta Atlassian, a hasło to token API. Token traktowany jako sekret.

Konsekwencje.

- Integracja działa niezależnie od SSO/Google.
- Token można unieważnić bez zmiany hasła konta.
- Docelowo token powinien być przechowywany jako secure parameter lub zmienna środowiskowa, aby nie trafił do eksportu przepływu.

## ADR-003: Schemat JSON generowany z pełnej odpowiedzi API

Kontekst. Akcja "Przeanalizuj dane JSON" wymaga schematu opisującego strukturę odpowiedzi. 

Schemat wygenerowano z pełnej, rzeczywistej odpowiedzi endpointu (`/users/1`).

Konsekwencje.

- Wszystkie pola odpowiedzi dostępne jako tokeny dynamiczne.

## ADR-004: API v2 (tekst) zamiast v3 (ADF) w pierwszej iteracji

Kontekst. Jira REST API v3 wymaga pola `description` w formacie ADF (Atlassian Document Format, struktura JSON). API v2 przyjmuje opis jako zwykły tekst.

Decyzja. Pierwsza iteracja korzysta z API v2 (`/rest/api/2/issue`).

Konsekwencje.

- Prostsze, czytelne body i szybsza droga do działającego rozwiązania.
- Migracja na v3/ADF możliwa jako osobna iteracja.
