# Architecture

The widget service is composed of three components — the API gateway — the widget engine — and the storage layer — which work together to provide widget management capabilities.

The API gateway, which is responsible for receiving incoming HTTP and gRPC requests from clients and which routes those requests to the appropriate handler based on the path and method, can scale horizontally by adding more replicas behind the load balancer.

It is important to note that the storage layer uses Postgres for durable state and Redis for ephemeral caching.
