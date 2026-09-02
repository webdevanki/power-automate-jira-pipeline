# Dokumentacja wymagań

Wymagania funkcjonalne i niefunkcjonalne rozwiązania oraz ich pokrycie w kolejnych iteracjach. Sposób realizacji (kroki przepływu) opisuje README w sekcji "Architektura", decyzje projektowe plik [adr.md](adr.md).

## 1. Interesariusze

| Rola | Odpowiedzialność |
|---|---|
| Pracownik (zgłaszający) | Rejestruje incydent na liście SharePoint |
| Zespół deweloperski | Odbiorca zgłoszeń w Jirze |
| Administrator procesu | Odbiorca powiadomień o błędach automatyzacji |
| Właściciel procesu | Monitoruje przebieg i kompletność zgłoszeń |

## 2. Wymagania funkcjonalne

| ID | Wymaganie | Priorytet |
|---|---|---|
| FR-01 | Pracownik rejestruje incydent (tytuł, opis, priorytet) w źródle danych | Wysoki |
| FR-02 | Utworzenie rekordu automatycznie uruchamia przepływ | Wysoki |
| FR-03 | Przepływ pobiera metadane zgłaszającego z zewnętrznego REST API | Średni |
| FR-04 | Przepływ parsuje odpowiedź JSON do postaci pozwalającej użyć pól w kolejnych krokach | Wysoki |
| FR-05 | Przepływ tworzy zgłoszenie w Jirze z mapowaniem pól rekordu (Summary, Description, Priority) | Wysoki |
| FR-06 | Po utworzeniu zgłoszenia przepływ ustawia status rekordu źródłowego na `Registered in Jira` | Średni |
| FR-07 | W razie błędu wywołania Jiry administrator otrzymuje powiadomienie e-mail z logiem i kodem statusu | Wysoki |
| FR-08 | Opis zgłoszenia jest wzbogacony o dane zgłaszającego z API (`name`, `email`) | Niski |

## 3. Wymagania niefunkcjonalne

| ID | Wymaganie |
|---|---|
| NFR-01 | Nieudane wywołanie API nie zatrzymuje procesu bez powiadomienia (wzorzec Try/Catch) |
| NFR-02 | Dostęp do pól JSON jest null-safe, brak pola nie przerywa przepływu |
| NFR-03 | Każde zgłoszenie ma odpowiednik w Jirze i ślad w źródle danych |

## 4. Pokrycie wymagań

Status odzwierciedla realny stan implementacji. Iteracje: v1 rdzeń integracji, v2 wzbogacony opis, v3 priorytet i zwrotna aktualizacja statusu, v4 obsługa błędów.

| ID | Status |
|---|---|
| FR-01 | Gotowe (v1) |
| FR-02 | Gotowe (v1) |
| FR-03 | Gotowe (v1) |
| FR-04 | Gotowe (v1) |
| FR-05 | Gotowe (v1), mapowanie priorytetu od v3 |
| FR-06 | Gotowe (v3) |
| FR-07 | Gotowe (v4) |
| FR-08 | Gotowe (v2) |
| NFR-01 | Gotowe (v4) |
| NFR-02 | Gotowe (v1) |
| NFR-03 | Gotowe (v3) |

## 5. Mapowanie statusów (SharePoint i Jira)

Oba systemy opisują ten sam proces z różnych perspektyw, więc statusy nie są identyczne.

| Perspektywa | System | Statusy |
|---|---|---|
| Zgłaszający pracownik | SharePoint | `New`, `Registered in Jira`, `In Progress`, `Done` |
| Zespół deweloperski | Jira | `To Do`, `In Progress`, `In Review`, `Done` |

Reguła przy utworzeniu zgłoszenia:

| Zdarzenie | SharePoint | Jira |
|---|---|---|
| Pracownik tworzy zgłoszenie | `New` | brak |
| Przepływ tworzy zgłoszenie w Jira | `Registered in Jira` | `To Do` |

Spośród statusów SharePoint (`New`, `Registered in Jira`, `In Progress`, `Done`) przepływ ustawia automatycznie wyłącznie `Registered in Jira`. `New` nadaje zgłaszający przy tworzeniu rekordu; `In Progress` i `Done` to zmiany ręczne lub przyszła synchronizacja zwrotna Jira do SharePoint (sekcja 6). Automatyzacja nigdy nie ustawia `In Progress` ani `Done`.

`Registered in Jira` to status kontrolny automatyzacji, nie stan pracy zespołu (patrz [ADR-006](adr.md)).

## 5a. Mapowanie priorytetu

Nazwy priorytetów pokrywają się w obu systemach, przepływ przekazuje wartość bez tłumaczenia.

| SharePoint | Jira |
|---|---|
| `Low` | `Low` |
| `Medium` | `Medium` |
| `High` | `High` |

## 6. Poza zakresem

- Eskalacja SLA i przypominanie o zaległych zgłoszeniach.
- Dwukierunkowa synchronizacja statusów Jira do źródła danych.
- Uwierzytelnianie użytkownika końcowego w warstwie Power Apps.
