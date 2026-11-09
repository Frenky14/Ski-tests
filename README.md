# Ski tests ⛷️

Ski test event and test center data used by a Shopify page.

This repository hosts the JSON data, not the storefront itself. It stays public so the page can fetch the data without GitHub authentication.

## Data

[`ski-data.json`](./ski-data.json) contains two top-level arrays:

| Key | Contents |
| --- | --- |
| `tests` | Ski test events, including location, dates, and a note or link. |
| `centers` | Test center locations. |

Country labels include English, German, Italian, French, Czech, and Slovenian translations.

## Public JSON endpoint

```text
https://raw.githubusercontent.com/Frenky14/Ski-tests/main/ski-data.json
```

Keep the repository name, branch, file path, and field names stable: the Shopify page may depend on them.

## Updating the data

1. Edit `ski-data.json`, preserving its existing structure and language keys.
2. Check that the file is valid JSON and that event dates use `YYYY-MM-DD`.
3. Commit the changes to `main`.
4. Check the Shopify page after any caching has refreshed.

## Public data only

Do not add passwords, API tokens, customer information, or other private data to this repository.
