# Shop Platform: Shared Project Spec (v0.1, Milestone 1)

## Product
Professional e-commerce platform for a medium-sized business.
Customer mobile app (Flutter) + backend (Django REST Framework). Staff use Django Admin.
Languages: Arabic and English (RTL support). Single store, single currency (pending decision).

## Roles
Customer (mobile app). Staff/Admin (Django Admin only).

## Stack
Flutter/Dart, Material 3, BLoC/Cubit, Clean Architecture, GoRouter, Dio, GetIt, Freezed.
Django, DRF, PostgreSQL, JWT (access + refresh), OpenAPI/Swagger. Redis/Celery only if justified.
Git + GitHub, Docker Compose, GitHub Actions.
Exact dependency versions: verified at scaffold time and recorded in docs/DECISIONS.md. Do not guess versions.

## Repository
shop-platform/ { mobile/ (M1), backend/ (M2), integration/ (M3), docs/, .github/ (M3),
docker-compose.yml (M3), .gitignore, README.md }
See docs/TEAM_OWNERSHIP.md for the file ownership matrix.

## Architecture rules
- API base: /api/v1/. All lists paginated: {count,next,previous,results}.
- Money: decimal strings ("129.90") + currency. Prices and totals are computed only by the server.
- Client sends product/variant IDs and quantities, never prices.
- Order creation: single DB transaction, select_for_update on stock, Idempotency-Key header required.
- OrderItem snapshots name, SKU, unit price at purchase time.
- Payment status changes only via signature-verified webhooks or server-side verification.
- Order status and payment status are separate. Transitions enforced in one backend service.
- Order status: pending_payment, paid, processing, shipped, delivered, cancelled, refunded.
- Payment status: initiated, authorized, paid, failed, refunded.
- Auth: email+password, short-lived access JWT, rotating refresh JWT. Customers can access only their own data.
- Localization: Accept-Language ar|en; localized catalog text.

## Error format
{ "error": { "code": "<snake_case>", "message": "<human text>", "details": { ... } } }
Codes: validation_error, unauthenticated, permission_denied, not_found, out_of_stock, price_changed,
coupon_invalid, payment_failed, conflict, rate_limited, server_error.

## Endpoint groups (/api/v1/)
auth (register, login, token/refresh, logout, password/change), me, categories, products, cart (+items, +coupon),
wishlist, addresses, checkout/preview, orders (+cancel), orders/{number}/payments, payments/webhooks/{provider},
health, schema/Swagger. Authoritative detail: docs/API_CONTRACT.md (authored by M2, approved by M1).

## Entities
User, Address, Category, Product, ProductVariant, ProductImage, StockMovement, Cart, CartItem, WishlistItem,
Coupon, CouponRedemption, Order, OrderItem, OrderStatusHistory, Payment, Shipment.
Authoritative detail: docs/DATABASE_DESIGN.md (authored by M2, approved by M1).

## Git workflow
main = stable releases; develop = integrated; feature/flutter-*, feature/backend-*, feature/integration-*.
Each member commits to own feature branches and opens PRs into develop. No force-pushes to shared branches.
No member edits files they do not own without an agreed task recorded in docs/DECISIONS.md.

## Milestones
1 Foundation; 2 Auth; 3 Catalog; 4 Cart+Wishlist; 5 Checkout+Orders+Payment; 6 Business management; 7 Release.

## Status of business decisions
Open: market/currency, payment methods/gateway, shipping model, tax, guest checkout, variants, coupon types,
returns/refunds, notifications, password reset/email verification. Do not invent these. Ask the leader (via the user).

## Reporting
Every member ends each work session with the handoff report format defined in their prompt.
Never claim a command ran, a test passed, or a push happened unless it actually did and you saw the output.