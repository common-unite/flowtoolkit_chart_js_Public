# Performance

Guidance for keeping the chart fast when working with large record collections.

## Pass only the records and fields you need

The chart processes whatever Flow gives it. The single biggest factor in render time is the size of the source collection. Use upstream **Get Records** filters and field selection to keep the input lean. Three thousand records with five fields each renders fast; thirty thousand records with thirty fields each will not.

There's no built-in cap — if Flow hands the chart fifty thousand records, the chart tries to render fifty thousand records.

## What the chart does on its own to stay fast

- **Reference field labels** (e.g., `AccountId` → `Acme Corp`) are resolved with one network call against a small sampled subset of the records, not against every record. Two thousand opportunities pointing at twenty distinct accounts produces a request for only twenty record Ids.
- **Picklist labels** are resolved from the field describe with zero per-record network cost and full locale support (translated picklist values render correctly).
- **Boolean, Date, and Number fields** never trigger a network call — formatting is handled client-side via the field type and the running user's locale.

## Lightning Web Security

Make sure **Lightning Web Security** is enabled in your org (Setup → Session Settings → "Use Lightning Web Security for Lightning web components"). The legacy Locker Service implementation wraps every record in a heavyweight proxy that scales poorly — large record collections get exponentially slower. With LWS enabled, proxying is lightweight.

If your chart feels much slower in production than in a clean scratch org, this is the first thing to check.

## Practical sizing guidance

| Collection size | Expected experience |
|---|---|
| Up to ~500 records | Snappy — sub-second renders |
| 500–2,000 records | Noticeable but acceptable — sub-2-second renders typical |
| 2,000–10,000 records | Slower — consider whether you really need this many records on a Screen |
| Over 10,000 records | Reconsider the use case. The chart will render but the Flow Screen experience will suffer. |

## When the chart waits for data

The chart shows a brief loading spinner while its async dependencies resolve — Chart.js scripts loading on first visit, the source object's field metadata, and any Reference label resolution. The spinner ensures the chart never paints partial state. After cache warm-up, subsequent visits skip the spinner.
