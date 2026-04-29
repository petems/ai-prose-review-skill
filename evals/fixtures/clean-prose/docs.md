# Connection Pooling in pgbouncer

pgbouncer is a lightweight connection pooler for Postgres. It sits between application servers and the database, multiplexing many client connections onto a smaller set of upstream connections. This document explains the three pooling modes and when to pick each one.

## Why pool connections

A Postgres backend process consumes about 10MB of memory per connection. At a few hundred connections the cost is manageable. At a few thousand the database spends more time context-switching than serving queries. Pooling caps the upstream connection count and lets clients wait their turn.

## Pooling modes

pgbouncer offers three modes, each trading isolation for throughput.

### Session pooling

A client gets a dedicated upstream connection for the duration of its session. Connections return to the pool only when the client disconnects. This mode preserves all Postgres features (prepared statements, session variables, advisory locks) but offers little multiplexing benefit.

Use session pooling when the client is short-lived and connects only when it needs the database.

### Transaction pooling

A client gets an upstream connection for the duration of a single transaction. The connection returns to the pool on commit or rollback. This is the most common production mode, since web applications typically run short transactions.

Transaction pooling breaks features that span transactions: server-side prepared statements, `LISTEN`/`NOTIFY`, temporary tables, and session-scoped GUCs. Most application frameworks already disable these features or work around them.

### Statement pooling

A client gets an upstream connection for the duration of a single statement. The connection returns to the pool after each query. This mode squeezes the most throughput out of a small upstream pool but breaks any feature that spans more than one statement, including transactions themselves. Few applications can use it.

## Choosing a mode

Pick transaction pooling unless you have a specific reason not to. If your application uses session-scoped features, switch to session pooling and accept the lower multiplexing ratio. Statement pooling is rarely the right choice; it usually indicates that the connection limit is set too low for the workload.

## Sizing the pool

The upstream pool size should match the database's `max_connections` minus a margin for superuser sessions and replication slots. Set `default_pool_size` to that target divided by the number of databases pgbouncer fronts. Monitor `SHOW POOLS` to see waiting clients; sustained waits mean the pool is undersized.
