# druto-cors

[![Crates.io](https://img.shields.io/crates/v/druto-cors.svg)](https://crates.io/crates/druto-cors)
[![Documentation](https://docs.rs/druto-cors/badge.svg)](https://docs.rs/druto-cors)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](LICENSE-MIT)

Cross-Origin Resource Sharing (CORS) preflight negotiation and response header middleware for Druto.

Operating as an asynchronous onion middleware layer on top of `dhap` and `druto-router`, `druto-cors` handles both preflight `OPTIONS` requests and actual cross-origin HTTP requests in accordance with the [W3C CORS Specification](https://www.w3.org/TR/cors/) and [RFC 6454](https://datatracker.ietf.org/doc/html/rfc6454).

---

## Features

- **Preflight Interception & Fail-Closed Short-Circuiting**:
  - Automatically identifies preflight `OPTIONS` requests containing `Origin` and `Access-Control-Request-Method`.
  - Responds immediately with status `204 No Content` (configurable to `200 OK`) and terminates execution before downstream handlers or database connections execute.
  - Fails closed upon invalid preflight headers by returning empty headers per W3C specification.
- **Flexible Origin Negotiation (`OriginPolicy`)**:
  - `OriginPolicy::Any`: Public APIs (`*`). Automatically reflects origin when credentials are enabled.
  - `OriginPolicy::Exact`: Whitelist of explicit domains.
  - `OriginPolicy::Pattern`: Wildcard subdomain patterns (e.g. `https://*.example.com`).
  - `OriginPolicy::Dynamic`: Runtime closure predicate `Fn(&str, &HttpContext) -> bool` for multi-tenant domain resolution.
  - `OriginPolicy::Mirror`: Direct request origin reflection.
- **Path-Scoped Execution**:
  - Scopes CORS processing to designated path patterns (e.g. `["api/*", "webhooks/*"]`), bypassing unaffected routes with zero overhead.
- **Cache Poisoning Defense (`Vary` Headers)**:
  - Automatically manages `Vary: Origin`, `Vary: Access-Control-Request-Method`, and `Vary: Access-Control-Request-Headers` preventing cache poisoning across CDNs and reverse proxies.
- **Header Normalization**:
  - Automatically normalizes exposed headers, filtering out default simple response headers per specification.

---

## Installation

Add `druto-cors` to your `Cargo.toml`:

```toml
[dependencies]
druto-cors = "0.1.0"
```

---

## Quick Start

```rust
use druto_cors::{CorsConfig, CorsMiddleware, OriginPolicy};
use druto_router::{HttpContext, IntoResponse, Router};

async fn api_handler(ctx: &mut HttpContext) -> impl IntoResponse {
    "Hello from cross-origin API!"
}

fn app_router() -> Router {
    let cors_config = CorsConfig::builder()
        .allow_origin(OriginPolicy::pattern(["https://*.example.com"]))
        .allow_credentials(true)
        .max_age(300)
        .build();

    let mut router = Router::new();
    router.use_middleware(CorsMiddleware::new(cors_config));
    router.get("/api/data", api_handler);
    router
}
```

---

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.
