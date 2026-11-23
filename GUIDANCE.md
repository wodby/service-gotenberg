# Gotenberg on Wodby

What Wodby sets up for this Gotenberg service. It runs the official `gotenberg/gotenberg` image, an HTTP API that converts web pages, HTML, Markdown and office documents to PDF.

## How applications reach it

- Base URL: `http://<app service name>:3000`. No TLS and no authentication is configured; no token is generated.
- No catalog service declares a link to it, so no variable with its address is set automatically. Put the base URL into the application's own configuration or a variable of the application service, and read it from there.
- Conversions are multipart `POST` requests to a route per conversion type, for example `/forms/chromium/convert/url`, `/forms/chromium/convert/html` (with an `index.html` file) and `/forms/libreoffice/convert`. A client written for another PDF service does not work by changing only the host and port.

## Versions

- `8` is the default; `9` is offered next to it. The image, user, port `3000`, `/health` and `/version` are the same in both.
- The Chromium and LibreOffice conversion routes keep their paths in 9, but some form fields changed. `/forms/pdfengines/*` routes moved to `/forms/pdf/*`, and post-processing fields such as `merge`, `pdfa` or `metadata` became entries of a `stages` list. A removed route answers `404` and a moved field `400`, each naming the replacement. Check the application's client against Gotenberg's upgrade guide before switching it to 9.
- Gotenberg 9 no longer ships Java, Python, pdfcpu, QPDF, pdftk or ExifTool, so their vulnerabilities no longer show up for this image.

## Things that matter in an environment

- Gotenberg fetches the page itself. A URL passed for conversion must be reachable from the Gotenberg container: use an address that resolves inside the environment or a public one, and pass cookies or headers the page needs with the request.
- The service is stateless, has no volume and can run several replicas. Results are returned in the response; nothing is kept.
- The manifest sets no Gotenberg options: timeouts, API basic authentication and other settings are Gotenberg's own flags and environment variables, set on this service.

## Check the result

- `GET http://<app service name>:3000/health` answers `200` with the status of the conversion engines.
- `GET http://<app service name>:3000/version` returns the running version.

## Basic auth

Wodby protects this service's public domains with basic auth by default, because Gotenberg has no authentication of its own and fetches the URLs it is given. Each app service of it gets its own generated password, which members reveal on the environment's Endpoints > Auths page. Applications inside the environment call it directly and need no credentials; only clients outside the environment send the username and password.
