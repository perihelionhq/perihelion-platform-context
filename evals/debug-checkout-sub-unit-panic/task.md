# Checkout is panicking on some orders

On-call paged us: `PlaceOrder` in the checkout service is crashing for some customers since the storefront started selling low-priced accessories. Those customers can't complete checkout, so we need a fix today.

From the logs:

```
panic: one of the specified money values is invalid

goroutine 4183 [running]:
github.com/open-telemetry/opentelemetry-demo/src/checkout/money.Must(...)
	/app/money/money.go:73
main.(*checkout).PlaceOrder(0xc000214000, {0x1a2b3c0, 0xc0004d2a80}, 0xc0003b6e60)
	/app/main.go:358 +0x6f4
```

Two orders that failed, both in GBP:

* 2 × lens cleaning cloth (£0.45 each) and 1 × red-dot finder (£24.00)
* 2 × lens cap tether (£0.30 each)

An order with 1 × lens cleaning cloth and 1 × red-dot finder went through fine.

Find and fix the cause. Add a short note at `docs/incidents/checkout-panic.md` explaining what went wrong, for the incident review.
