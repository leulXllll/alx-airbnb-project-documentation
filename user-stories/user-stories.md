User Stories for Airbnb Clone Backend

This document translates the interactions from the Airbnb Clone backend use case diagram into user stories. Each story captures a core functionality, representing the perspective of key actors (Guest, Host, Admin) and aligning with the system’s features.
User Stories

    Register User
        As a new user, I want to register an account with my email and password so that I can access the platform as a guest, host, or admin.
        Use Case: Register User
        Database: User table (email, password_hash, role)

    Create Booking
        As a guest, I want to create a booking for an available property so that I can reserve it for my desired dates.
        Use Case: Create Booking, Check Property Availability
        Database: Booking table (booking_id, property_id, user_id, start_date, end_date)

    Process Payment
        As a guest, I want to securely process a payment for my booking so that I can confirm my reservation.
        Use Case: Process Payment
        Database: Payment table (payment_id, booking_id, amount, payment_method)

    List Property
        As a host, I want to list a new property with details like location and price so that guests can book it.
        Use Case: List Property
        Database: Property table (property_id, host_id, name, location, pricepernight)

    Handle Refund
        As an admin, I want to process a refund for a canceled booking so that guests receive their money back according to cancellation policies.
        Use Case: Handle Refund, Cancel Booking
        Database: Payment table (booking_id, amount)

    Cancel Booking
        As a guest, I want to cancel my booking so that I can change my travel plans without penalty, if allowed.
        Use Case: Cancel Booking
        Database: Booking table (status)
