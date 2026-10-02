# Add an order status tool to the shopping assistant

Customers ask the shopping assistant "where is my order?". Add an agent tool that looks up an order's shipping status.

* The storefront exposes `GET /api/orders/{orderId}/status`, which returns JSON like `{"orderId": "...", "status": "shipped", "trackingId": "..."}`. It returns 404 for an unknown order.
* Add the tool alongside the existing tools in `shared/tools.py` and register it with the agent in `src/agents/agents.py`.
* Add automated tests for the tool. The repository has no tests yet; add whatever minimal test setup you need.
