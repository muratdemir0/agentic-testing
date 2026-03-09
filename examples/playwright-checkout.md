# Example: Playwright Acceptance Test — Checkout Flow

This example demonstrates the `acceptance-test-writer` skill applied to a typical e-commerce checkout flow.

**Skill used:** `acceptance-test-writer`  
**Framework:** Playwright (TypeScript)  
**Scenario:** A returning customer adds a product to their cart, applies a coupon, and completes payment.

The test covers the happy path (successful purchase with coupon) and two failure paths (expired coupon, declined card).

---

## Test File

```typescript
import { test, expect } from '@playwright/test';
import { CartPage } from './pages/CartPage';
import { CheckoutPage } from './pages/CheckoutPage';
import { ProductPage } from './pages/ProductPage';

test.describe('Checkout flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('customer@example.com');
    await page.getByLabel('Password').fill('testpassword');
    await page.getByRole('button', { name: 'Sign in' }).click();
    await expect(page).toHaveURL('/dashboard');
  });

  test('user can complete checkout with a valid coupon code', async ({ page }) => {
    const product = new ProductPage(page);
    const cart = new CartPage(page);
    const checkout = new CheckoutPage(page);

    await test.step('add product to cart', async () => {
      await product.goto('wireless-headphones');
      await product.addToCart();
      await expect(cart.badge).toHaveText('1');
    });

    await test.step('apply valid coupon', async () => {
      await cart.goto();
      await cart.applyCoupon('SAVE10');
      await expect(cart.discountLine).toBeVisible();
      await expect(cart.discountLine).toContainText('10%');
    });

    await test.step('complete payment', async () => {
      await checkout.goto();
      await checkout.fillCardDetails({
        number: '4242 4242 4242 4242',
        expiry: '12/26',
        cvc: '123',
      });
      await checkout.submitOrder();
    });

    await test.step('confirm order placed', async () => {
      await expect(page).toHaveURL(/\/orders\/\d+\/confirmation/);
      await expect(page.getByRole('heading', { name: 'Order confirmed' })).toBeVisible();
      await expect(page.getByText('A confirmation email has been sent')).toBeVisible();
    });
  });

  test('user sees an error when applying an expired coupon', async ({ page }) => {
    const product = new ProductPage(page);
    const cart = new CartPage(page);

    await product.goto('wireless-headphones');
    await product.addToCart();
    await cart.goto();
    await cart.applyCoupon('EXPIRED50');

    await expect(page.getByRole('alert')).toContainText('This coupon has expired');
    await expect(cart.discountLine).not.toBeVisible();
  });

  test('user sees a declined message when card is rejected', async ({ page }) => {
    const product = new ProductPage(page);
    const cart = new CartPage(page);
    const checkout = new CheckoutPage(page);

    await product.goto('wireless-headphones');
    await product.addToCart();
    await cart.goto();
    await checkout.goto();
    await checkout.fillCardDetails({
      number: '4000 0000 0000 0002', // Stripe test card: always declined
      expiry: '12/26',
      cvc: '123',
    });
    await checkout.submitOrder();

    await expect(page.getByRole('alert')).toContainText('Your card was declined');
    await expect(page).not.toHaveURL(/\/confirmation/);
  });
});
```

---

## Page Object: CheckoutPage

```typescript
// pages/CheckoutPage.ts
import { Page } from '@playwright/test';

export class CheckoutPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/checkout');
  }

  async fillCardDetails(card: { number: string; expiry: string; cvc: string }) {
    await this.page.getByLabel('Card number').fill(card.number);
    await this.page.getByLabel('Expiry date').fill(card.expiry);
    await this.page.getByLabel('CVC').fill(card.cvc);
  }

  async submitOrder() {
    await this.page.getByRole('button', { name: 'Place order' }).click();
  }
}
```

---

## Notes

**Selector strategy:** All selectors use `getByRole`, `getByLabel`, and `getByText`. No CSS classes or arbitrary `data-testid` attributes appear in assertions. This makes the suite resilient to styling changes and DOM restructuring.

**Page Object Model:** `ProductPage`, `CartPage`, and `CheckoutPage` encapsulate navigation and interaction. Tests read as user stories, not DOM queries.

**Failure coverage:** The suite includes both an expired coupon path and a declined card path. These are not edge cases — they are expected user experiences the product must handle correctly. Omitting them leaves real risk undetected.

**Assertions focus on visible outcomes:** The tests assert what the user sees — a confirmation heading, an error alert, a URL pattern. They do not inspect database records, Redux state, or API responses. That is the correct scope for an acceptance test.

**Stripe test cards:** The declined card number `4000 0000 0000 0002` is a standard Stripe test card. Replace with your payment provider's equivalent test values.

> This example was generated using the `acceptance-test-writer` skill. For the corresponding output test on the receipt JSON produced by this flow, see [`go-approval-receipt.md`](go-approval-receipt.md).
