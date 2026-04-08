# Fakebook Web Crawler

## Approach

The crawler opens a fresh TLS connection per request (`Connection: close`), sends manually formatted HTTP/1.1 requests, and parses status lines, headers, and bodies. Chunked `Transfer-Encoding` and `Content-Length` responses are both supported. `Set-Cookie` headers are merged into a small cookie jar and sent back on subsequent requests.

Login loads `/accounts/login/?next=/fakebook/`, parses the POST form action and hidden fields (including the CSRF token), then POSTs `application/x-www-form-urlencoded` credentials. Redirects (302) and intermittent 503 responses are retried until a final status is obtained.

The crawl starts at `/fakebook/` and uses a FIFO frontier (queue) with a visited set keyed by a normalized URL (scheme, host with default port collapsed, path, query; fragment dropped). Only HTTPS URLs on the configured host and port are enqueued. HTTP 302 is followed on the same site; 403/404 skip the page; 503 retries with backoff.

Secret flags are matched with a regular expression against the assignment’s `<h3 class='secret_flag' …>FLAG: …</h3>` pattern.

## Challenges

- Correctly reading HTTP/1.1 bodies when the server uses chunked encoding versus a fixed content length.
- Preserving multiple `Set-Cookie` headers and submitting them together on later requests.
- Following redirects after POST without treating 503 retries as redirect budget.

## Testing

Run with your Northeastern username and Fakebook password from the course password page:

```bash
make
./crawler -s fakebook.khoury.northeastern.edu -p 443 YOUR_USERNAME YOUR_PASSWORD
```

The program prints five flag lines to stdout. Fill `secret_flags` with your five flags before submitting.
