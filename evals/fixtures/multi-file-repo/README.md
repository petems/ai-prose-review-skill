# widget-svc

widget-svc is a microservice that manages widgets. It exposes a REST API and a gRPC API, both backed by the same Postgres database.

Furthermore, the service publishes events to Kafka.
Furthermore, it integrates with the auth service for authorization.

## Quick start

```
docker compose up
```

Then visit http://localhost:8080/widgets.
