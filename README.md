# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In the Observer Pattern, a `Subscriber` is typically defined as an interface (or a trait in Rust) to allow multiple implementations for different types of subscribers. However, in the BambangShop case, a single model struct is sufficient because all subscribers share the same structure and behavior. Since the `Subscriber` struct only holds basic data such as `url` and `name`, there is no immediate need for a trait/interface. If in the future we need different types of subscribers, then implementing a `Subscriber` trait/interface would be beneficial. However, for now, a single struct is enough to keep the implementation simple and efficient.

2. In this tutorial, we store subscribers using `DashMap<String, DashMap<String, Subscriber>>`, which ensures that each `Subscriber` is uniquely identified by their `url` within a `product_type`. Using a `Vec<Subscriber>` instead of `DashMap` would not be an ideal solution because a `Vec` does not enforce uniqueness, meaning we would have to perform an O(n) search every time we insert or remove a subscriber to avoid duplicates. On the other hand, `DashMap` allows O(1) lookups, insertions, and deletions, making it more efficient. Additionally, `DashMap` is thread-safe and allows concurrent access without requiring manual locking, which would be necessary if we used a `Vec` inside a `Mutex` or `RwLock`. Since multiple parts of the application may need to access and modify subscriber data simultaneously, `DashMap` is the best choice for ensuring performance and correctness.

3. In this tutorial, we use `lazy_static!` to define `SUBSCRIBERS` as a global, static variable, making it accessible throughout the application. This already provides a Singleton-like behavior, as there is only one instance of `SUBSCRIBERS` shared across the program. However, simply using the Singleton pattern without `DashMap` would not be enough, because a Singleton alone does not provide thread safety. If we replaced `DashMap` with a `Mutex<HashMap>`, we would have to lock the entire structure every time we read or modify it, leading to performance bottlenecks in a multi-threaded environment. `DashMap`, on the other hand, provides built-in concurrency handling with sharded locking, making it more efficient for high-read, high-write scenarios. Because of this, we should continue using `DashMap` rather than replacing it with a traditional Singleton pattern.

#### Reflection Publisher-2
1. In the traditional Model-View-Controller (MVC) pattern, the Model is responsible for both data storage and business logic, but this can lead to tightly coupled code, making it harder to maintain and extend. By introducing a Service layer, we separate business logic from data representation, allowing for better modularity and reusability. Similarly, the Repository layer abstracts the data access logic, ensuring that the Model does not directly interact with the database or storage mechanisms. In this tutorial, the `NotificationService` handles subscription and unsubscription logic, while `SubscriberRepository` manages data storage using `DashMap`. This separation improves code organization, allows for easier unit testing, and makes future changes more manageable without affecting the core business logic.

2. If we only use the Model without separating business logic and data access into Service and Repository layers, the code would become significantly more complex and harder to maintain. In this case, the `Program`, `Subscriber`, and `Notification` models interact frequently, and if all business logic were directly inside these models, they would need to handle data retrieval, updates, validation, and business rules all in one place. This would lead to tight coupling, making it difficult to introduce changes without modifying multiple parts of the code. For example, if the notification logic were embedded directly inside the `Subscriber` model, any changes to the notification system would require modifying the subscriber model itself. By keeping responsibilities separate, as we have done in this tutorial, we make the code more scalable, testable, and maintainable.

3. Postman is an essential tool for testing API endpoints because it allows us to send HTTP requests without needing to write a client application. In this tutorial, Postman helps verify that the `subscribe` and `unsubscribe` endpoints work as expected, ensuring that subscribers are correctly added and removed. The Postman collection given by Lecturer/Dosen includes specific API endpoints such as `http://localhost:8000/notification/subscribe/APPLIANCES`, which allows testing different product types dynamically. Additionally, it includes essential requests for creating, listing, and deleting products, as well as receiving notifications in the Receiver app.<br> One of the most useful features of Postman is the ability to save and organize requests into collections, making it easy to reuse them during development. Additionally, environment variables could be used in Postman to dynamically switch between different ports (`8000` for the Publisher/Main and `8001` for the Receiver app), making testing even more efficient. These features make Postman a powerful tool not only for this tutorial but also for real-world software projects, where API testing is crucial.

#### Reflection Publisher-3
1. In this tutorial, we implement the Push model of the Observer Pattern, where the publisher (main app) actively sends notifications to all subscribers (receiver apps) whenever an event occurs. This can be seen in the `NotificationService::notify` function, where a `Notification` payload is immediately created and sent to each subscriber using an HTTP POST request. The subscribers do not need to request or check for updates; instead, they receive updates automatically whenever a product is created, deleted, or promoted. This approach ensures that subscribers always receive the latest information without having to constantly check for changes.

2. If we were to use the Pull model instead, the subscribers would periodically request updates from the publisher rather than receiving them automatically. One advantage of this approach is that it reduces unnecessary notifications, as subscribers only request updates when they need them, which can reduce network traffic. Additionally, subscribers can control their own update frequency, allowing them to check for new data at their own pace.<br> However, the Pull model also has several disadvantages in this tutorial case. Since product updates happen dynamically, subscribers may miss important updates if they do not check frequently enough. This would make real-time notifications unreliable. Additionally, having multiple subscribers frequently polling the publisher could create unnecessary load on the server, leading to performance issues. Because BambangShop requires real-time updates, the Push model is the better choice, as it ensures that every subscriber is notified as soon as a relevant product change occurs.

3. If we remove multi-threading from the notification process, each subscriber update would be executed sequentially, meaning the main application would wait for each HTTP request to complete before moving on to the next subscriber. This would significantly slow down the notification process, especially if there are many subscribers or if some take longer to respond. In the worst case, a slow or unresponsive subscriber could block the entire process, delaying notifications to all other subscribers.<br> With multi-threading, each notification is handled in a separate thread, allowing multiple notifications to be sent concurrently. This improves performance and responsiveness, as the main application does not have to wait for each individual request to complete before continuing execution. By using `thread::spawn`, we ensure that the notification system can scale efficiently without being bottlenecked by slow subscribers.