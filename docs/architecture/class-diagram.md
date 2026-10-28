# Class Diagram

Covers the SQLAlchemy ORM models defined in `booking_system_backend/models.py`,
the Pydantic API schemas in `booking_system_backend/schemas.py`, and the
`BookingStatus` values inferred from the service and seed layers. Shows all
attributes, their types, and the relationships between classes.

```mermaid
classDiagram

    %% ── BookingStatus (implicit string enum) ──────────────────────────
    class BookingStatus {
        <<enumeration>>
        booked
        cancelled
        completed
    }

    %% ── SQLAlchemy ORM models (models.py) ──────────────────────────────
    class User {
        +int user_id PK
        +str name
        +str email
    }

    class Flight {
        +int flight_id PK
        +str origin
        +str destination
        +str departure_time
        +str arrival_time
        +int price
        +int seats_available
    }

    class Booking {
        +int booking_id PK
        +int user_id FK
        +int flight_id FK
        +str status
        +str booking_time
    }

    %% ── Pydantic schemas (schemas.py) ──────────────────────────────────
    class FlightOut {
        +int flight_id
        +str origin
        +str destination
        +str departure_time
        +str arrival_time
        +int price
        +int seats_available
    }

    class BookingRequest {
        +int user_id
        +str name
        +int flight_id
    }

    class BookingOut {
        +int booking_id
        +int user_id
        +int flight_id
        +str status
        +str booking_time
    }

    class UserRegistration {
        +str name
        +str email
    }

    class UserOut {
        +int user_id
        +str name
        +str email
    }

    class ErrorResponse {
        +bool success
        +str error
        +str error_code
        +str details
    }

    %% ── Relationships ──────────────────────────────────────────────────

    %% ORM foreign-key associations
    Booking --> User : user_id FK
    Booking --> Flight : flight_id FK

    %% Status is constrained to BookingStatus values
    Booking ..> BookingStatus : status (string)

    %% Pydantic ↔ ORM correspondence (from_attributes = True)
    FlightOut  ..> Flight  : serialises
    BookingOut ..> Booking : serialises
    UserOut    ..> User    : serialises

    %% BookingRequest carries a user name (used to create/lookup User)
    BookingRequest ..> User   : resolves name → user_id
    BookingRequest ..> Flight : references flight_id
```
