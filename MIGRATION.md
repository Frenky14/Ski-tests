# Switch Shopify to the combined dataset

`ski-data.json` contains `tests` (from `db.json`) and `centers` (from `db2.json`). Records, IDs, translations, and dates are unchanged. The old test dataset is archived at `archive/dbtest.json` and is not part of the live data.

The old `db.json` and `db2.json` files are temporarily retained so existing pages keep working. They are snapshots, not automatically synchronized copies. After switching all consumers, remove them and maintain only `ski-data.json`.

## 1. Publish the data

Push `ski-data.json` to the GitHub repository's `main` branch before changing Shopify. Local file changes alone do not update the public URL.

## 2. Change the root element's data attributes

Remove `data-tests-url` and `data-centers-url`, and add:

```html
data-source-url="https://raw.githubusercontent.com/Frenky14/Ski-tests/main/ski-data.json"
```

Keep `id`, `class`, `data-locale`, and the rest of the HTML/CSS unchanged.

## 3. Replace the two loader functions

In the supplied script, replace both `loadTests` and `loadCenters` with this single function. Keep all other helpers, rendering, and event listeners unchanged.

```javascript
const loadData = async () => {
  const response = await fetch(root.dataset.sourceUrl, { cache: 'no-store' });
  if (!response.ok) throw new Error(`Failed to load ski data: ${response.status}`);
  const data = await response.json();

  if (!Array.isArray(data.tests) || !Array.isArray(data.centers)) {
    throw new Error('Invalid ski data: expected tests and centers arrays');
  }

  const normalizedData = data.tests.map((row) => ({
    ...row,
    startDate: row.startDate || row.date || '',
    endDate: row.endDate || row.startDate || row.date || ''
  }));
  const today = new Date();
  today.setHours(0, 0, 0, 0);

  state.tests = normalizedData
    .filter((row) => {
      const finalDate = parseIsoDate(row.endDate) || parseIsoDate(row.startDate);
      return finalDate && finalDate >= today;
    })
    .sort((a, b) => parseIsoDate(a.startDate) - parseIsoDate(b.startDate));

  state.centers = data.centers;
  state.centers.sort((a, b) =>
    countryLabel(a).localeCompare(countryLabel(b), locale) ||
    (a.skiResort || '').localeCompare(b.skiResort || '', locale)
  );

  populateCountries('tests');
  render('tests');
  populateCountries('centers');
  render('centers');
};
```

## 4. Replace the final loading call

Replace:

```javascript
Promise.all([loadTests(), loadCenters()]).catch((error) => console.error(error));
```

with:

```javascript
loadData().catch((error) => console.error(error));
```

## 5. Check the published page

- Confirm the browser Network panel shows one request to `ski-data.json`, with a successful response.
- Check both tabs, search, country filters, translated country names, date sorting, and the Hinterstoder email link.
- Existing behavior is preserved: past events and events with `TBD` dates remain hidden. Possible duplicate events and notes containing a URL plus extra text have not been changed in this migration.

The Shopify snippet is not stored in this repository, so its changes must be applied in Shopify separately.
