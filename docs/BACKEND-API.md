# Backend API

Routes that `tshirt-backend` adds to Medusa, plus the standard Medusa routes the storefront depends on.
Every `/store/*` call needs the header `x-publishable-api-key`. Every `/admin/*` call needs an admin
session or bearer token. Errors are `{ "message": "..." }`.

`BASE` is the backend origin, `http://localhost:9000` in development.

## Storefront (custom)

### `PUT /store/designs/:designId/:side/:kind`

Stores one file of a saved design. The storefront calls it three times per side.

| Part | Rule |
|---|---|
| `designId` | 8-64 characters of `A-Z a-z 0-9 -` (the storefront sends a UUID) |
| `side` | `front` or `back` |
| `kind` | `png` (print file, max 40 MB), `jpg` (preview, max 10 MB), `json` (editor scene, max 5 MB) |
| Body | the raw bytes. The Content-Type header is ignored; the kind decides it |

The bytes are checked against the kind (PNG or JPEG signature, or a JSON object). The file is named
`designs/{designId}/{side}.{kind}` by the server.

- `201 { "url": "https://.../designs/{designId}/{side}.{kind}" }`
- `400` invalid reference, or the bytes are not what the kind says
- `413` too large (also enforced while streaming, so a false `Content-Length` does not help)
- `429` more than 60 uploads in 10 minutes from one address (`Retry-After` says when to retry)
- `502` the bucket refused or was unreachable

### `GET /store/designs/:designId/:side/json`

Returns the editor scene JSON, so a design can be reopened without any CORS rule on the bucket.
Images are not served through this route: `png` and `jpg` answer `404`.

### `POST /store/order-lookup`

Order tracking without an account.

```json
{ "order": "#12", "email": "customer@example.com" }
```

`order` is the number shown to the customer (`#12` or `12`) or the full id (`order_01...`).

- `200 { "order": { ... } }` with `id`, `display_id`, `created_at`, `currency_code`, `items[]`
  (`title`, `quantity`, `unit_price`, `thumbnail`, `designs[] {side, image_url}`, `garment`),
  `subtotal`, `shipping_total`, `total`, `shipping_address`, `payment {method, status}`, `note`,
  `jobs[] {id, order_item_id, side, status, tracking_number}`
- `400` malformed input
- `404` no order matches. **A wrong number and a wrong email get the same answer**, so the endpoint cannot
  be used to find out which orders exist
- `429` more than 10 attempts in 10 minutes from one address

Not returned: print-shop file URLs, internal notes, the email. Totals are summed from the lines.

## Storefront (standard Medusa)

Used in this order by the storefront:

| Step | Call |
|---|---|
| Regions, products | `GET /store/regions`, `GET /store/products?region_id=...&fields=+variants.calculated_price` |
| Cart | `POST /store/carts`, `POST /store/carts/:id/line-items` (design in `metadata`), `POST /store/carts/:id/line-items/:line_id` |
| Checkout | `GET /store/shipping-options?cart_id=`, `POST /store/carts/:id` (email, addresses, `metadata.note`), `POST /store/carts/:id/shipping-methods`, `POST /store/payment-collections`, `POST /store/payment-collections/:id/payment-sessions` (`pp_system_default` = cash on delivery), `POST /store/carts/:id/complete` |

Completing the cart emits `order.placed`; a subscriber creates one print job per printed side.
Line item `metadata` carries `garment` and `designs[]` (`side`, `png_url`, `json_url`, `preview_url`, `placement`);
the shapes are `CartLineItemMetadata` and `DesignAsset` in `@tshirt-platform/shared`.

## Admin: print jobs

| Method | Route | Action |
|---|---|---|
| GET | `/admin/print-orders?status=&order_id=&offset=&limit=` | List (`limit` up to 100). Returns `{ print_jobs, count, offset, limit }` |
| GET | `/admin/print-orders/:id` | One job |
| POST | `/admin/print-orders/:id` | Body `{ status, tracking_number?, notes? }`. Moves the job along its status flow |
| POST | `/admin/print-orders/:id/cancel` | Cancel (only before shipping) |
| GET | `/admin/print-orders/:id/package` | Zip for the print shop: transparent PNG at print size and 300 DPI, `spec.json`, work order, proof |

Status flow (a job cannot enter production before the customer's proof is approved):

```
pending -> proof_approved -> processing -> shipped -> delivered
   \             \               \
    +-------------+---------------+--> cancelled
```

An invalid move answers `400` with the reason. The package route only downloads design files from the
origins in `DESIGN_FILE_ORIGINS`, because a customer can edit the URLs in their cart.

## Admin: preview templates

Proxied to the render service (`tshirt-render`), which has no accounts of its own. Uploads are streamed.

| Method | Route |
|---|---|
| GET / POST | `/admin/mockups` (POST is multipart: `image`, `name`) |
| GET / DELETE | `/admin/mockups/:id` |
| GET | `/admin/mockups/:id/image` |
| PUT | `/admin/mockups/:id/mask`, `/admin/mockups/:id/occlusion` (multipart) |
| PUT | `/admin/mockups/:id/quad` (JSON) |
| PUT | `/admin/mockups/:id/quad/fit` (JSON `{print_width_mm, print_height_mm, top_offset_mm, garment_length_mm}`): puts the print at its true size and proportions, centred, hanging below the shoulder line |
| PUT | `/admin/mockups/:id/mask/outline`, `/admin/mockups/:id/occlusion/outline` (JSON `{outlines: [[[x,y]...]], refine}`, corners in 0..1): hand-drawn garment area / parts in front of the print, for photos that are not on a plain background. The edge is snapped to the photo |
| DELETE | `/admin/mockups/:id/occlusion` |

`product.metadata.print_config.mockups` is `{ front: string[], back: string[] }`: template ids in the order the
storefront shows them (an older single id still reads as a list of one). The photo shows one reference size; its
body length turns millimetres into pixels for `quad/fit`.

## Health

`GET /health` answers `200` when the server is up (open, no key).

## Checking it

`pnpm smoke` (in `tshirt-backend`) plays the whole storefront flow against a running backend:
upload, cart, COD checkout, order lookup, print job. It places a real order for `smoke@example.com`,
so use development or staging.

```bash
BACKEND_URL=http://localhost:9000 PUBLISHABLE_KEY=pk_... pnpm smoke
```
