# Fakebook Web Crawler


The crawler opens a TLS connection per request (`Connection: close`), sends manually formatted HTTP/1.1 requests, and parses status lines, headers, and bodies. Chunked `Transfer-Encoding` and `Content-Length` responses are both supported. `Set-Cookie` headers are merged into a small cookie jar and sent back.

Login loads `/accounts/login/?next=/fakebook/`, parses the POST  action and hidden fields, then POSTs `application/x-www-form-urlencoded` credentials. Redirects (302) and intermittent 503 responses are retried until a final status is obtained.

The crawl starts at `/fakebook/` and uses a FIFO approach (queue) with a visited set. This has a normalized URL (scheme, host with default port). Only HTTPS URLs on the configured host and port are enqueued. HTTP 302 is followed on the same site; 403/404 skip the page; 503 retries with backoff.

Secret flags are matched with a regular expression against the assignment’s `<h3 class='secret_flag' …>FLAG: …</h3>` pattern.

## Challenges

- Correctly reading HTTP/1.1 bodies when the server uses chunked encoding versus a fixed content length.
- Preserving multiple `Set-Cookie` headers and submitting them together on later requests.
- Following redirects after POST without treating 503 retries as redirect budget.

## Testing

```bash
make
./crawler -s fakebook.khoury.northeastern.edu -p 443 YOUR_USERNAME YOUR_PASSWORD
```
