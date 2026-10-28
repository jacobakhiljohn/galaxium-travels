# Booking Flow Sequence Diagram

Shows the complete request/response flow for a user booking a flight, traced
across the React frontend (`booking_system_frontend/src/services/api.ts`), the
FastAPI REST layer (`booking_system_backend/server.py`), the booking service
(`booking_system_backend/services/booking.py`), and the SQLite database.
Includes all error branches: flight not found, no seats, user not found, and
name mismatch.

```mermaid
sequenceDiagram
    actor User

    participant UI   as Frontend<br/>(React / api.ts)
    participant API  as Backend REST<br/>(FastAPI server.py)
    participant SVC  as BookingService<br/>(services/booking.py)
    participant DB   as SQLite<br/>(booking.db)

    %% ── 1. Browse flights ────────────────────────────────────────────
    User->>UI: open flight search
    UI->>API: GET /flights
    API->>SVC: flight.list_flights(db)
    SVC->>DB: SELECT * FROM flights
    DB-->>SVC: rows
    SVC-->>API: List[FlightOut]
    API-->>UI: 200 List[FlightOut]
    UI-->>User: display flight list

    %% ── 2. Resolve user identity ──────────────────────────────────────
    User->>UI: enter name + email, click "Book"
    UI->>API: GET /user?name=&email=
    API->>DB: SELECT * FROM users WHERE name=… AND email=…
    alt user found
        DB-->>API: UserOut
        API-->>UI: 200 UserOut (user_id)
    else user not found
        DB-->>API: null
        API-->>UI: 200 ErrorResponse (USER_NOT_FOUND)
        UI-->>User: show error — user must register first
    end

    %% ── 3. Book flight ────────────────────────────────────────────────
    UI->>API: POST /book {user_id, name, flight_id}
    API->>SVC: booking.book_flight(db, user_id, name, flight_id)

    SVC->>DB: SELECT * FROM flights WHERE flight_id=…
    alt flight not found
        DB-->>SVC: null
        SVC-->>API: ErrorResponse (FLIGHT_NOT_FOUND)
        API-->>UI: 200 ErrorResponse
        UI-->>User: show error
    else no seats available
        DB-->>SVC: Flight (seats_available = 0)
        SVC-->>API: ErrorResponse (NO_SEATS_AVAILABLE)
        API-->>UI: 200 ErrorResponse
        UI-->>User: show error
    else flight OK
        DB-->>SVC: Flight (seats_available ≥ 1)
        SVC->>DB: SELECT * FROM users WHERE user_id=… AND name=…
        alt user_id exists but name mismatch
            DB-->>SVC: null (id found, name wrong)
            SVC-->>API: ErrorResponse (NAME_MISMATCH)
            API-->>UI: 200 ErrorResponse
            UI-->>User: show error
        else user_id not found
            DB-->>SVC: null
            SVC-->>API: ErrorResponse (USER_NOT_FOUND)
            API-->>UI: 200 ErrorResponse
            UI-->>User: show error
        else user OK
            DB-->>SVC: User row
            SVC->>DB: UPDATE flights SET seats_available = seats_available - 1
            SVC->>DB: INSERT INTO bookings (user_id, flight_id, status="booked", booking_time)
            DB-->>SVC: new Booking row
            SVC-->>API: BookingOut
            API-->>UI: 200 BookingOut
            UI-->>User: show booking confirmation
        end
    end
```
