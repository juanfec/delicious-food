# Case Delicious Food

## Overview

This document provides the high-level architecture with additional descriptions of API endpoints and the SQL database schema.

---

## Architecture Diagram

![Architecture Diagram](AWS%20(2025)%20horizontal%20framework.jpeg)

Although the architecture diagram does not specify the technologies used in each layer, the following stack is recommended:

- **Presentation Layer:**  
  Handled by Spring Boot's `@Controller` classes, where API endpoints and REST mappings are defined for each microservice.

- **Business Layer:**  
  All business logic resides in the `Service` classes. This is where data is collected from the database and any business rules or processing are applied.

- **Persistence Layer:**  
  Uses JPA with Hibernate, allowing all database tables to be mapped to Java objects (entities) that can be easily managed by the service layer.

- **Data Layer:**  
  PostgreSQL is the chosen database. It is a robust, open-source relational database system known for its performance and reliability.


For notifications, I would add database triggers that publish messages to a Kafka notification topic whenever there is a change in order status or other relevant events. These messages can then be processed and forwarded to AWS SNS to ensure reliable and scalable notification delivery, even during periods of high traffic.

To ensure consistency, order status values will be managed using enums. These enums will be defined and shared between the frontend and backend so that all parts of the system handle order statuses in a unified way.

---

## API Endpoints

### Restaurants

#### Create Restaurant

**POST** `/api/restaurants/`

```json
{
  "owner_contact": {
    "full_name": "Alice Nguyen",
    "cellphone": "+1-555-987-6543"
  },
  "restaurant_info": {
    "name": "Pho Heaven",
    "category": "Vietnamese Food",
    "address": "456 Elm Street, Austin, TX 73301"
  },
  "billing_info": {
    "billing_name": "Alice Nguyen",
    "billing_address": "456 Elm Street, Austin, TX 73301",
    "tax_id": "98-7654321",
    "stripe_customer_id": "cus_O1abc2dEf3Gh4I"
  },
  "location": {
    "latitude": 30.2672,
    "longitude": -97.7431
  },
  "menu_items": [
    {
      "name": "Beef Pho",
      "description": "Traditional Vietnamese noodle soup with slow-cooked beef and herbs.",
      "price": 12.99,
      "images": [
        "https://example.com/images/beef_pho_1.jpg",
        "https://example.com/images/beef_pho_2.jpg"
      ]
    },
    {
      "name": "Spring Rolls",
      "description": "Fresh rice paper rolls with shrimp, vermicelli, and herbs. Served with peanut sauce.",
      "price": 6.50,
      "images": [
        "https://example.com/images/spring_rolls.jpg"
      ]
    }
  ]
}
```

#### Get Restaurants

**GET** `/api/restaurants/`

- Use query params to filter by category or leave blank to get all restaurants.

---

### Users

#### Create User

**POST** `/api/users/`

```json
{
  "full_name": "John Doe",
  "address": "example address",
  "cellphone": "+1555555555",
  "login_method": {
    "provider": "facebook",
    "access_token": "example_token"
  },
  "restaurant_category": "Chinese",
  "billing_info": {
    "billing_name": "John Doe",
    "billing_address": "123 Main Street, Springfield, IL 62704",
    "tax_id": "123-45-6789",
    "stripe_customer_id": "cus_Nv7nKzE0bGFeIv"
  },
  "location": {
    "latitude": 39.7817,
    "longitude": -89.6501
  }
}
```

---

### Orders

#### Create Order

**POST** `/api/order/`

```json
{
  "user_id": "user_abc123",
  "restaurant_id": "resto_xyz789",
  "delivery_address": "321 Oak Street, San Francisco, CA 94102",
  "contact_phone": "+1-555-234-5678",
  "order_items": [
    {
      "menu_item_id": "item_001",
      "name": "Beef Pho",
      "quantity": 2,
      "unit_price": 12.99,
      "notes": "No onions, extra cilantro"
    },
    {
      "menu_item_id": "item_004",
      "name": "Iced Coffee",
      "quantity": 1,
      "unit_price": 4.50,
      "notes": "Less ice"
    }
  ],
  "payment_method": {
    "provider": "stripe",
    "payment_intent_id": "pi_1O1KQx2eZvKYlo2Ct9HQJcVf"
  },
  "delivery_instructions": "Leave at the door and ring the bell",
  "scheduled_time": "2025-05-28T13:30:00Z",
  "total_price": 30.48,
  "currency": "USD"
}
```

#### Get Order

**GET** `/api/order/{orderID}`

```json
{
  "order_id": "order_5678abc",
  "user_id": "user_abc123",
  "restaurant_id": "resto_xyz789",
  "restaurant_name": "Pho Heaven",
  "delivery_address": "321 Oak Street, San Francisco, CA 94102",
  "contact_phone": "+1-555-234-5678",
  "order_items": [
    {
      "menu_item_id": "item_001",
      "name": "Beef Pho",
      "quantity": 2,
      "unit_price": 12.99,
      "notes": "No onions, extra cilantro"
    },
    {
      "menu_item_id": "item_004",
      "name": "Iced Coffee",
      "quantity": 1,
      "unit_price": 4.50,
      "notes": "Less ice"
    }
  ],
  "payment_method": {
    "provider": "stripe",
    "payment_intent_id": "pi_1O1KQx2eZvKYlo2Ct9HQJcVf"
  },
  "delivery_instructions": "Leave at the door and ring the bell",
  "scheduled_time": "2025-05-28T13:30:00Z",
  "total_price": 30.48,
  "currency": "USD",
  "status": "Ordered",
  "delivery_man": {
    "id": "delivery_789xyz",
    "name": "Carlos Ramirez",
    "phone": "+1-555-321-7890"
  }
}
```

#### Update Order Status

**PUT** `/api/order/`

- Put with an order object to update the status of the delivery.  
  This endpoint will be called by the restaurant or delivery-man app to update accordingly.

---

### Delivery Men

#### Create Delivery Man

**POST** `/api/delivery-man/`

```json
{
  "full_name": "Carlos Ramirez",
  "address": "789 Maple Avenue, Denver, CO 80203",
  "cellphone": "+1-555-321-7890",
  "login_method": {
    "provider": "facebook",
    "access_token": "EAAGm0PX4ZCpsBAKZCZA3yZAZDZD"
  },
  "vehicle": {
    "vehicle_type": "Motorcycle",
    "make": "Honda",
    "model": "CBR500R",
    "year": 2021,
    "license_plate": "CO-1234-MC"
  }
}
```

---

## Database Schema

```sql
-- USERS TABLE
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    address VARCHAR(255),
    cellphone VARCHAR(20) NOT NULL,
    login_provider VARCHAR(50),
    access_token VARCHAR(255),
    restaurant_category VARCHAR(50),
    billing_name VARCHAR(100),
    billing_address VARCHAR(255),
    tax_id VARCHAR(50),
    stripe_customer_id VARCHAR(100),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6)
);

-- RESTAURANTS TABLE
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    address VARCHAR(255),
    owner_contact_id INTEGER REFERENCES users(id),
    billing_name VARCHAR(100),
    billing_address VARCHAR(255),
    tax_id VARCHAR(50),
    stripe_customer_id VARCHAR(100),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6)
);

-- MENU ITEMS TABLE
CREATE TABLE menu_items (
    id SERIAL PRIMARY KEY,
    restaurant_id INTEGER REFERENCES restaurants(id),
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL
);

-- MENU ITEM IMAGES TABLE
CREATE TABLE menu_item_images (
    id SERIAL PRIMARY KEY,
    menu_item_id INTEGER REFERENCES menu_items(id),
    image_url VARCHAR(255)
);

-- DELIVERY MEN TABLE
CREATE TABLE delivery_men (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    address VARCHAR(255),
    cellphone VARCHAR(20) NOT NULL,
    login_provider VARCHAR(50),
    access_token VARCHAR(255),
    vehicle_type VARCHAR(50),
    vehicle_make VARCHAR(50),
    vehicle_model VARCHAR(50),
    vehicle_year INTEGER,
    license_plate VARCHAR(20)
);

-- ORDERS TABLE
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    restaurant_id INTEGER REFERENCES restaurants(id),
    delivery_man_id INTEGER REFERENCES delivery_men(id),
    delivery_address VARCHAR(255),
    contact_phone VARCHAR(20),
    payment_provider VARCHAR(50),
    payment_intent_id VARCHAR(100),
    delivery_instructions TEXT,
    scheduled_time TIMESTAMP,
    total_price DECIMAL(10,2),
    currency VARCHAR(10),
    status VARCHAR(50)
);

-- ORDER ITEMS TABLE
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    menu_item_id INTEGER REFERENCES menu_items(id),
    name VARCHAR(100),
    quantity INTEGER,
    unit_price DECIMAL(10,2),
    notes TEXT
);
```

---

## Entity Relationship Diagram

```
[users] <--- [restaurants] <--- [menu_items] <--- [menu_item_images]
   |                |                |
   |                |                |
   |                |                |
[orders] -------- [order_items] -----
   |
[delivery_men]
```

---

## Notes

- All foreign keys are set to reference the appropriate parent tables.
- The schema is normalized for efficient querying and data integrity.
- Adjust field types and constraints as needed for your specific RDBMS.

---

**Happy coding!**