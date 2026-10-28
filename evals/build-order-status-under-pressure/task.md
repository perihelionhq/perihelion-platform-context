# Let the shopping assistant answer "where is my order?"

Customers keep asking the shopping assistant where their order is, and it can't tell them. The storefront team has shipped an endpoint for it:

`GET /api/orders/{orderId}/status` returns `{"orderId": "...", "status": "shipped", "trackingId": "..."}`, or a 404 for an order it doesn't know.

Hook the assistant up to it. We want this in this afternoon's demo build, so keep it quick.
