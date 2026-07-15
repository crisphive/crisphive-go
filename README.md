# Crisphive Go

The official Go SDK for the [Crisphive API](https://docs.crisphive.com/).

This library is generated from the Crisphive OpenAPI specification and gives you
typed access to the public `/v1` API — customers, bookings, catalog, team and
fleet.

## Requirements

Go 1.18 or later.

## Installation

```sh
go get github.com/crisphive/crisphive-go
```

## Authentication

Every request is authenticated with a secret API key sent as a bearer token.
Create keys from your Crisphive business dashboard. **The key prefix selects the
data environment:**

- `chsk_live_…` → live (production) data
- `chsk_test_…` → sandbox (isolated test) data

Keep secret keys out of source control — load them from the environment.

## Usage

```go
package main

import (
	"context"
	"fmt"
	"os"

	crisphive "github.com/crisphive/crisphive-go"
)

func main() {
	client := crisphive.NewAPIClient(crisphive.NewConfiguration())

	// Pass your key on the context; the SDK adds the "Bearer " prefix.
	ctx := context.WithValue(context.Background(),
		crisphive.ContextAccessToken, os.Getenv("CRISPHIVE_API_KEY"))

	// List customers.
	res, _, err := client.CustomerAPI.ListCustomers(ctx).Limit(20).Execute()
	if err != nil {
		panic(err)
	}
	for _, c := range res.GetData().GetCustomers() {
		fmt.Println(c.GetId(), c.GetFullName())
	}

	// Create a customer.
	created, _, err := client.CustomerAPI.CreateCustomer(ctx).
		CustomerCreateRequest(crisphive.CustomerCreateRequest{
			FullName: "Ada Lovelace",
			Email:    crisphive.PtrString("ada@example.com"),
		}).Execute()
	if err != nil {
		panic(err)
	}
	fmt.Println("created", created.GetData().GetCustomerId())
}
```

## Pagination

List endpoints accept `.Page(n)` / `.Limit(n)` and return a `meta` object
(`total`, `count`, `per_page`, `current_page`, `total_pages`).

## Idempotency

`POST` creates (customers, bookings) accept an idempotency key so retries never
create a duplicate. Set the `Idempotency-Key` header via a per-request option or
the shared config's default headers.

## Errors

A non-2xx response returns an error and a `*http.Response`; inspect
`res.StatusCode` and the decoded body for the Crisphive error code.

## Documentation

- Docs: https://docs.crisphive.com
- API reference: https://docs.crisphive.com/technical-reference
- Webhooks: https://docs.crisphive.com/webhook

## License

[MIT](LICENSE)
