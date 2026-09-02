# Dokumentacja wymagań

Wymagania funkcjonalne i niefunkcjonalne rozwiązania oraz ich pokrycie w kolejnych iteracjach. Sposób realizacji (kroki przepływu) opisuje README w sekcji "Architektura v1", decyzje projektowe plik [adr.md](adr.md).

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
| FR-05 | Przepływ tworzy zgłoszenie w Jirze z mapowaniem pól rekordu (Summary, Description) | Wysoki |
| FR-06 | Po utworzeniu zgłoszenia status rekordu źródłowego zmienia się na `In Progress` | Średni |
| FR-07 | W razie błędu wywołania Jiry administrator otrzymuje powiadomienie z logiem i kodem statusu | Wysoki |
| FR-08 | Opis zgłoszenia jest wzbogacony o pola z API (`name`, `email`, `company`) | Niski |

## 3. Wymagania niefunkcjonalne

| ID | Wymaganie |
|---|---|
| NFR-01 | Nieudane wywołanie API nie zatrzymuje procesu bez powiadomienia (Try/Catch) |
| NFR-02 | Dostęp do pól JSON jest null-safe, brak pola nie przerywa przepływu |
| NFR-03 | Każde zgłoszenie ma odpowiednik w Jirze i ślad w źródle danych |

## 4. Pokrycie wymagań

Status odzwierciedla realny stan implementacji.

| ID | Status |
|---|---|
| FR-01 | Gotowe (v1) |
| FR-02 | Gotowe (v1) |
| FR-03 | Gotowe (v1) |
| FR-04 | Gotowe (v1) |
| FR-05 | Gotowe (v1) |
| FR-06 | Planowane |
| FR-07 | Planowane |
| FR-08 | Planowane |
| NFR-01 | Planowane |
| NFR-02 | Gotowe (v1) |
| NFR-03 | Częściowo, pełne pokrycie po FR-06 |

## 5. Mapowanie statusów (SharePoint i Jira)

Oba systemy opisują ten sam proces z różnych perspektyw, więc statusy nie są identyczne.

| Perspektywa | System | Statusy |
|---|---|---|
| Zgłaszający pracownik | SharePoint | `New`, `In Progress`, `Done` |
| Zespół deweloperski | Jira | `To Do`, `In Progress`, `In Review`, `Done` |

Reguła przy utworzeniu zgłoszenia:

| Zdarzenie | SharePoint | Jira |
|---|---|---|
| Pracownik tworzy zgłoszenie | `New` | brak |
| Przepływ tworzy zgłoszenie w Jira | `In Progress` (planowane) | `To Do` |

## 6. Poza zakresem

- Eskalacja SLA i przypominanie o zaległych zgłoszeniach.
- Dwukierunkowa synchronizacja statusów Jira do źródła danych.
- Uwierzytelnianie użytkownika końcowego w warstwie Power Apps.
