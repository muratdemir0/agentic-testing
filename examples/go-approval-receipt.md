# Example: Approval Test — Receipt JSON (Go)

This example demonstrates the `approval-test-writer` skill applied to a receipt serializer in Go.

**Skill used:** `approval-test-writer`  
**Language:** Go  
**Framework:** `go-approval-tests`  
**Scenario:** A billing service generates a receipt after a successful order. The approval test verifies the serialized JSON does not change unintentionally.

> **Note:** This example uses Go. The `approval-test-writer` skill is **language-agnostic** — it generates approval tests in your project's existing language and test framework. Go is used here as an illustration. The same pattern applies in TypeScript, Python, Ruby, Java, C#, and others. See the skill file for a full language compatibility table.

---

## Test File

```go
package billing_test

import (
	"encoding/json"
	"regexp"
	"testing"
	"time"

	approvals "github.com/approvals/go-approval-tests"
	"github.com/example/billing"
	"github.com/google/uuid"
)

func TestReceiptJSON(t *testing.T) {
	// Arrange: construct a receipt using fixed, deterministic values.
	// Using fixed values at construction time is cleaner than scrubbing after the fact
	// when you control the test data.
	receipt := billing.Receipt{
		ID:        uuid.MustParse("a1b2c3d4-0000-0000-0000-000000000001"),
		OrderID:   "ORD-00001",
		CreatedAt: time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC),
		Customer: billing.Customer{
			Name:  "Alice Johnson",
			Email: "alice@example.com",
		},
		Items: []billing.LineItem{
			{SKU: "HDPH-001", Name: "Wireless Headphones", Qty: 1, UnitPrice: 8999},
			{SKU: "CASE-002", Name: "Carry Case", Qty: 1, UnitPrice: 1499},
		},
		Discount: billing.Discount{
			Code:       "SAVE10",
			Percentage: 10,
			Amount:     1050,
		},
		Subtotal: 10498,
		Tax:      945,
		Total:    10393,
		Currency: "USD",
	}

	// Act: serialize to indented JSON for a readable diff.
	output, err := json.MarshalIndent(receipt, "", "  ")
	if err != nil {
		t.Fatalf("failed to marshal receipt: %v", err)
	}

	// Scrub: normalize any remaining unstable values.
	// In this test we injected fixed values above, so scrubbing is a safety net.
	// In tests against production code that generates its own IDs or timestamps,
	// scrubbing is the primary normalization mechanism.
	scrubbed := scrubReceipt(string(output))

	// Assert: compare against the approved file.
	// On first run this will fail and write a .received.txt file for your review.
	approvals.VerifyString(t, scrubbed)
}

// scrubReceipt replaces fields that are inherently unstable across runs.
// Use this when you cannot inject deterministic values at construction time.
func scrubReceipt(s string) string {
	s = scrubTimestamps(s)
	s = scrubUUIDs(s)
	return s
}

// scrubTimestamps replaces ISO 8601 timestamps with a stable placeholder.
func scrubTimestamps(s string) string {
	re := regexp.MustCompile(`\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z`)
	return re.ReplaceAllString(s, "<timestamp>")
}

// scrubUUIDs replaces UUID-formatted values with a stable placeholder.
func scrubUUIDs(s string) string {
	re := regexp.MustCompile(
		`[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}`,
	)
	return re.ReplaceAllString(s, "<uuid>")
}
```

---

## Approved File

Save as `TestReceiptJSON.approved.txt` in the same directory as the test. This file is committed to version control.

```json
{
  "id": "<uuid>",
  "order_id": "ORD-00001",
  "created_at": "<timestamp>",
  "customer": {
    "name": "Alice Johnson",
    "email": "alice@example.com"
  },
  "items": [
    {
      "sku": "HDPH-001",
      "name": "Wireless Headphones",
      "qty": 1,
      "unit_price": 8999
    },
    {
      "sku": "CASE-002",
      "name": "Carry Case",
      "qty": 1,
      "unit_price": 1499
    }
  ],
  "discount": {
    "code": "SAVE10",
    "percentage": 10,
    "amount": 1050
  },
  "subtotal": 10498,
  "tax": 945,
  "total": 10393,
  "currency": "USD"
}
```

---

## First-Run Approval Workflow

1. Run `go test ./billing/...`
2. The test fails on the first run and writes `TestReceiptJSON.received.txt`
3. Open `TestReceiptJSON.received.txt` and read it — this is the output you are about to approve
4. If the output looks correct, rename it: `mv TestReceiptJSON.received.txt TestReceiptJSON.approved.txt`
5. Commit `TestReceiptJSON.approved.txt` to version control
6. Run the test again — it should now pass

## Re-Approval Workflow

When the receipt format changes legitimately (e.g., a `tax_rate` field is added):

1. The test fails and writes a new `.received.txt`
2. Open a diff: `diff TestReceiptJSON.approved.txt TestReceiptJSON.received.txt`
3. Review every line of the diff — confirm the change is intentional
4. Rename the received file to approved
5. Commit the updated approved file with a clear commit message explaining what changed

**Never approve without reviewing.** The diff is the test result. If you approve without reading it, the test provides no value.

---

## Notes

**Fixed values vs. scrubbing:** When you control the test data, inject deterministic values directly into the struct. This produces the cleanest approved files. Use `scrubReceipt` as a fallback when you are testing code that generates its own IDs or timestamps internally.

**Approved file in version control:** The `.approved.txt` file is a first-class artifact. It documents exactly what the receipt should look like. Treat changes to it with the same care as changes to source code.

**Indented JSON:** `json.MarshalIndent` produces a human-readable diff. Never use `json.Marshal` in an approval test — a single-line JSON blob produces an unreadable diff.

**Ordering:** If `Items` order is not semantically meaningful in your domain, sort by SKU before serializing. Unstable ordering creates false positives. Document the sort decision in the test.

**This is not a Go-only pattern.** The identical workflow applies in every language — serialize, scrub, compare against approved file, review diffs deliberately. See the `approval-test-writer` skill for library recommendations per language.

> This example was generated using the `approval-test-writer` skill. For the corresponding behavioral test on the checkout flow that triggers this receipt, see [`playwright-checkout.md`](playwright-checkout.md).
