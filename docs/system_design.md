# Sofibor Inventory and Process Control

This document proposes a design for the Sofibor inventory control and process management system built with Laravel.

## Use Cases by Role

### Administrators (Davi, Gabriel, Marcelo)
- Manage user accounts and permissions.
- Register component entries and exits.
- Track welding operations and movements to chemical bath.
- Edit or cancel separation requests (SC).
- Record bath treatments and destinations (SamyParts or Cachoeira).
- Generate and view all reports.

### Celma
- Read-only access to consult any component or piece.
- View whether a piece was separated, sent to bath, returned, or forwarded to SamyParts/Cachoeira.

### Cabral
- Same read access as Celma.
- Create SC (Separation Requests) with: date, SC number, separation date, SO code, internal codes, component descriptions, quantity, and notes.

### Maurício
- Same read access as Celma.
- Respond to SC with: separated quantity, weight, bath treatment, bath destination, send date, and notes.

## Database Schema (MySQL or PostgreSQL)

```
users
  id PK
  name
  email
  password
  role (administrator, viewer, requester, responder)
  timestamps

components
  id PK
  code_internal
  description
  stock_quantity
  created_at
  updated_at

pieces
  id PK
  component_id FK -> components.id
  welded (boolean)
  ready_for_bath (boolean)
  current_status (separated | in_bath | returned)
  bath_company (nullable: samy | cachoeira)
  created_at
  updated_at

separation_requests (sc)
  id PK
  sc_number
  request_date
  separation_date
  so_code
  requester_id FK -> users.id
  observations
  created_at
  updated_at

separation_items
  id PK
  sc_id FK -> separation_requests.id
  component_id FK -> components.id
  quantity

sc_responses
  id PK
  sc_id FK -> separation_requests.id
  responder_id FK -> users.id
  quantity_separated
  weight
  bath_treatment
  bath_company
  send_date
  observations

baths
  id PK
  piece_id FK -> pieces.id
  treatment
  company
  send_date
  return_date
  observations

inventory_movements
  id PK
  component_id FK -> components.id
  piece_id FK -> pieces.id (nullable)
  type (entry | exit)
  quantity
  date
  notes
```

## Interface Proposal
- **Dashboard** summarizing pending SCs, pieces in bath, and recent movements.
- **Responsive** layout using Laravel with [Tailwind CSS](https://tailwindcss.com/).
- **Navigation**: Components, Pieces, Separation Requests, Baths, Reports, Admin.
- **Forms** for SC creation and response with dynamic component selection.
- **Tables** with search/filter to consult stock or SC status.
- **Alert system** (Laravel notifications) to remind when baths should return or when stock is low.

## Useful Reports
- **Movement History**: filter by component or SC to see all entries and exits.
- **Pending Separations**: list SCs awaiting response.
- **Current Status**: search any piece and view last operation (separated, in bath, returned, destination).

## Laravel + XAMPP Setup
1. Install XAMPP and enable MySQL and Apache.
2. Install [Composer](https://getcomposer.org/).
3. Create the project:
   ```bash
   composer create-project laravel/laravel sofibor
   ```
4. Copy the project into XAMPP's `htdocs` and configure `.env` with database credentials.
5. Run migrations to create the tables:
   ```bash
   php artisan migrate
   ```
6. Serve the application with
   ```bash
   php artisan serve
   ```
7. Access `http://localhost:8000` in your browser.

## Automation and AI
- Use Laravel's scheduling to send email alerts when bath return dates approach.
- Consider integrating a chatbot (e.g., BotMan) for quick queries on stock status.

