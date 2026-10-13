# Inter-Service Communication Standard: RabbitMQ (Tier 2 Delta)

## 1. Purpose
Defines the message envelopes, exchange topologies, routing conventions, and consumer error handling for asynchronous RabbitMQ event-driven communication.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1, AND
- `inter_service_comm: rabbitmq` is selected in Step 7.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Base Standard | Inherits correlation tracing from `standards/07-communication/rest.md`. |
| Cross-Cutting | Message payloads embed `correlationId` to maintain end-to-end tracing across async boundaries. |

---

## 4. Contract (Delta from REST)

This standard replaces synchronous HTTP calls with asynchronous message brokerage, **EXCEPT**:

1. **Broker Connection:**
   - Broker URI MUST be validated via typed configuration: `RABBITMQ_URI`.
   - Client library MUST be `amqplib` (or `@golevelup/nestjs-rabbitmq` for NestJS).
2. **Standard Message Envelope:**
   - All published events MUST adhere to this exact envelope:
     ```json
     {
       "id": "<uuidv4>",
       "type": "<entity>.<action>",
       "source": "<emitting-service-name>",
       "timestamp": "<ISO-8601-UTC>",
       "correlationId": "<incoming-or-generated-uuid>",
       "data": { }
     }
     ```
3. **Naming Conventions:**
   - Exchanges: `<domain>.events` (type: `topic`).
   - Routing Keys: `<entity>.<action>` (e.g. `order.created`, `user.registered`).
   - Queues: `<consuming-service>.<entity>.<action>`.
4. **Dead-Letter Handling & Idempotency:**
   - Failed messages exceeding retry counts MUST be routed to a dead-letter exchange (`dlx`) and dead-letter queue (`dlq`).
   - Consumers MUST be idempotent: processing the same message `id` twice MUST NOT produce duplicate side-effects.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// RabbitMQ event publication contract
export async function publishDomainEvent<T>(channel: Channel, routingKey: string, payload: T, correlationId: string) {
  const envelope = {
    id: crypto.randomUUID(),
    type: routingKey,
    source: 'order-service',
    timestamp: new Date().toISOString(),
    correlationId,
    data: payload
  };
  channel.publish('orders.events', routingKey, Buffer.from(JSON.stringify(envelope)), {
    persistent: true,
    headers: { 'x-correlation-id': correlationId }
  });
}
```

---

## 6. Agent Output Checklist
- [ ] Dependencies include `amqplib` or NestJS RabbitMQ integration.
- [ ] Message envelope includes `id`, `type`, `source`, `timestamp`, `correlationId`, and `data`.
- [ ] DLQ exchange and queue configurations declared.

---

## 7. Common Mistakes
- **Publishing Raw Unwrapped JSON:** Omitting the message envelope, making correlation ID tracking impossible.
- **Non-Idempotent Consumers:** Directly incrementing counters or charging cards without checking for previous message processing.
