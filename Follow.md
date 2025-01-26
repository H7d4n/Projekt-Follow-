
# Dokumentacja projektu Follow

## Opis projektu
Follow — to aplikacja internetowa do zarządzania wydarzeniami oraz subskrypcji wydarzeń innych użytkowników. Użytkownicy mogą tworzyć konta, publikować swoje wydarzenia, subskrybować wydarzenia innych użytkowników, a także otrzymywać powiadomienia przed rozpoczęciem i po zakończeniu wydarzeń.

## Główne technologie:
- **Node.js**: Platforma serwerowa do obsługi zapytań.
- **Express.js**: Framework do routingu i zarządzania zapytaniami HTTP.
- **EJS**: Silnik szablonów do generowania dynamicznych stron HTML.
- **Axios**: Klient do wykonywania zapytań HTTP.
- **PostgreSQL**: Relacyjna baza danych do przechowywania informacji o użytkownikach, wydarzeniach i subskrypcjach.
- **AWS Lambda**: Obliczenia serverless do obsługi operacji logowania/wylogowania użytkowników i wysyłania powiadomień.
- **AWS SNS**: Usługa powiadomień do dostarczania wiadomości użytkownikom.
- **AWS RDS**: Zarządzana baza danych PostgreSQL.

## Architektura

### Główne moduły aplikacji:

#### Uwierzytelnianie i autoryzacja:
- Używana jest **AWS Lambda** do obsługi rejestracji i logowania do kont. Użytkownicy mogą tworzyć konta i logować się, wysyłając zapytania przez API.
- AWS Cognito zarządza tokenami i weryfikacją tożsamości użytkowników.

#### Praca z wydarzeniami:
- Użytkownicy mogą tworzyć, edytować i usuwać wydarzenia poprzez interfejs. Te operacje są obsługiwane przez **Express.js** i zapisywane w bazie danych **PostgreSQL**.
- Subskrypcje wydarzeń innych użytkowników są zarządzane poprzez ścieżki API.

#### Powiadomienia:
- **AWS Lambda** okresowo wywołuje **AWS SNS** do wysyłania powiadomień do subskrybowanych użytkowników:
  - Dzień przed rozpoczęciem wydarzenia.
  - Natychmiast po rozpoczęciu wydarzenia.

#### Praca z bazą danych:
- **PostgreSQL**, uruchomiona na **AWS RDS**, przechowuje dane o użytkownikach, wydarzeniach i subskrypcjach.

### Baza danych:
Baza danych jest zaprojektowana z wykorzystaniem następujących tabel:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL
);

CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    author_id INTEGER REFERENCES users(id)
);

CREATE TABLE subscriptions (
    user_id INTEGER REFERENCES users(id),
    event_id INTEGER REFERENCES events(id),
    PRIMARY KEY (user_id, event_id)
);
```

---

## Integracje AWS

### AWS Lambda
Używana do wykonywania funkcji serverless:

#### Funkcje rejestracji i logowania:
- Przetwarzają zapytania użytkowników przez REST API.
- Komunikują się z AWS Cognito w celu zarządzania tokenami uwierzytelniającymi.

#### Funkcje powiadomień:
- Okresowo wywoływane za pomocą Amazon CloudWatch Events.
- Sprawdzają bazę danych w poszukiwaniu wydarzeń wymagających powiadomień.
- Wysyłają powiadomienia za pomocą AWS SNS.

### AWS SNS
- Używana do wysyłania powiadomień:
  - E-mail lub SMS dzień przed wydarzeniem.
  - Powiadomienia o rozpoczęciu wydarzenia.

---

## Główne API

### Rejestracja użytkownika
**Endpoint:** `POST /api/auth/register`

**Przykładowe zapytanie:**
```json
{
  "name": "Imię użytkownika",
  "email": "email@example.com",
  "password": "hasło"
}
```

### Logowanie użytkownika
**Endpoint:** `POST /api/auth/login`

**Przykładowe zapytanie:**
```json
{
  "email": "email@example.com",
  "password": "hasło"
}
```

### Tworzenie wydarzenia
**Endpoint:** `POST /api/events`

**Przykładowe zapytanie:**
```json
{
  "title": "Nazwa wydarzenia",
  "description": "Opis wydarzenia",
  "start_time": "2025-01-01T10:00:00Z",
  "end_time": "2025-01-01T12:00:00Z"
}
```

### Subskrypcja wydarzenia
**Endpoint:** `POST /api/events/:id/subscribe`
