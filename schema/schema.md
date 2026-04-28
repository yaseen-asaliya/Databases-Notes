# Sakila Database Schema

## Entity Relationship Diagram

```
                        ┌──────────────┐
                        │   language   │
                        │──────────────│
                        │ language_id  │◄──────────────────┐
                        │ name         │                   │
                        └──────────────┘                   │
                                                           │
┌─────────────┐     ┌──────────────────────────────────────┴──┐     ┌──────────────┐
│    actor    │     │                  film                    │     │   category   │
│─────────────│     │──────────────────────────────────────────│     │──────────────│
│ actor_id    │     │ film_id                                  │     │ category_id  │
│ first_name  │     │ title                                    │     │ name         │
│ last_name   │     │ description                              │     └──────┬───────┘
└──────┬──────┘     │ release_year                             │            │
       │            │ language_id (FK)                         │            │
       │            │ rental_duration                          │            │
       │            │ rental_rate                              │            │
       │            │ length                                   │            │
       │            │ replacement_cost                         │            │
       │            │ rating                                   │            │
       │            └──────────┬──────────────────────────┬───┘            │
       │                       │                          │                │
       │   ┌───────────────────┘              ┌───────────┘                │
       │   │                                  │                            │
       │  ┌▼────────────┐           ┌─────────▼──────┐                    │
       │  │  film_text  │           │ film_category  │◄───────────────────┘
       │  │─────────────│           │────────────────│
       │  │ film_id(FK) │           │ film_id (FK)   │
       │  │ title       │           │ category_id(FK)│
       │  │ description │           └────────────────┘
       │  └─────────────┘
       │
       │  ┌──────────────┐
       └──► film_actor   │
          │──────────────│
          │ actor_id(FK) │
          │ film_id (FK) │
          └──────┬───────┘
                 │
          ┌──────▼──────────────────┐
          │       inventory         │
          │─────────────────────────│
          │ inventory_id            │
          │ film_id (FK)            │
          │ store_id (FK)           │
          └──────────┬──────────────┘
                     │
          ┌──────────▼──────────────┐       ┌──────────────────┐
          │         rental          │       │     customer     │
          │─────────────────────────│       │──────────────────│
          │ rental_id               │       │ customer_id      │
          │ rental_date             │       │ store_id (FK)    │
          │ inventory_id (FK)       │       │ first_name       │
          │ customer_id (FK)        ├──────►│ last_name        │
          │ return_date             │       │ email            │
          │ staff_id (FK)           │       │ address_id (FK)  │
          └──────────┬──────────────┘       │ active           │
                     │                      └────────┬─────────┘
          ┌──────────▼──────────────┐               │
          │         payment         │       ┌────────▼─────────┐
          │─────────────────────────│       │     address      │
          │ payment_id              │       │──────────────────│
          │ customer_id (FK)        │       │ address_id       │
          │ staff_id (FK)           │       │ address          │
          │ rental_id (FK)          │       │ district         │
          │ amount                  │       │ city_id (FK)     │
          │ payment_date            │       │ postal_code      │
          └─────────────────────────┘       │ phone            │
                                            └────────┬─────────┘
┌──────────────────┐                                │
│      store       │                       ┌────────▼─────────┐
│──────────────────│                       │       city       │
│ store_id         │                       │──────────────────│
│ manager_staff_id │                       │ city_id          │
│ address_id (FK)  │                       │ city             │
└────────┬─────────┘                       │ country_id (FK)  │
         │                                 └────────┬─────────┘
┌────────▼─────────┐                               │
│      staff       │                      ┌─────────▼────────┐
│──────────────────│                      │     country      │
│ staff_id         │                      │──────────────────│
│ first_name       │                      │ country_id       │
│ last_name        │                      │ country          │
│ address_id (FK)  │                      └──────────────────┘
│ store_id (FK)    │
│ email            │
│ active           │
└──────────────────┘
```

---

## Tables Summary

| Table          | Rows (approx) | Purpose                                      |
|----------------|---------------|----------------------------------------------|
| `film`         | 1,000         | Movie catalog                                |
| `actor`        | 200           | Actors                                       |
| `film_actor`   | 5,462         | Many-to-many: films ↔ actors                 |
| `film_category`| 1,000         | Many-to-many: films ↔ categories             |
| `film_text`    | 1,000         | Fulltext search mirror of film title/desc    |
| `category`     | 16            | Genre categories                             |
| `language`     | 6             | Film languages                               |
| `inventory`    | 4,581         | Physical copies of films per store           |
| `rental`       | 16,044        | Each rental transaction                      |
| `payment`      | 16,049        | Payments per rental                          |
| `customer`     | 599           | Store customers                              |
| `staff`        | 2             | Store employees                              |
| `store`        | 2             | Physical store locations                     |
| `address`      | 603           | Addresses for customers/staff/stores         |
| `city`         | 600           | Cities                                       |
| `country`      | 109           | Countries                                    |

---

## Key Relationships

```
film ──< film_actor >── actor          (a film has many actors, an actor appears in many films)
film ──< film_category >── category    (a film belongs to categories)
film ──< inventory                     (a film has many physical copies)
inventory ──< rental                   (a copy can be rented many times)
rental >── customer                    (a rental belongs to one customer)
rental ──< payment                     (a rental has payments)
customer >── address >── city >── country
store >── address
staff >── store
```