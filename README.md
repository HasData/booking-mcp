# Booking.com MCP Server

<!-- mcp-name: com.hasdata/booking -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client two read-only Booking.com tools. Search stays by destination and dates with rich filters, and read a single property in full, all as structured JSON, with nothing to host.

It reads public property pages on Booking.com that a signed-out visitor can see.

**1,000 free credits every month, no card required**, which is 100 Booking.

```
https://mcp.hasdata.com/api/mcp?apis=booking
```

[![Glama score](https://glama.ai/mcp/servers/HasData/booking-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/booking-mcp)
[![tool contract](https://github.com/HasData/booking-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/booking-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-2-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/booking-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/booking-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-booking-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-booking-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp), free to create with no card, and the free tier covers 100 calls a month at the 10-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/booking-mcp` on npm and `hasdata-booking-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/api/mcp?apis=booking` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http booking "https://mcp.hasdata.com/api/mcp?apis=booking" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/api/mcp?apis=booking` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/booking-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "booking": {
      "command": "npx",
      "args": ["-y", "@hasdata/booking-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "booking": {
      "command": "uvx",
      "args": ["hasdata-booking-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "booking": {
      "url": "https://mcp.hasdata.com/api/mcp?apis=booking",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "booking": {
      "serverUrl": "https://mcp.hasdata.com/api/mcp?apis=booking",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "booking": {
      "type": "http",
      "url": "https://mcp.hasdata.com/api/mcp?apis=booking",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 10 credits.

> Search Booking.com for hotels in Paris from September 15 to 18 for two adults, and give me the ten best-reviewed under $700 for the stay.

*One call, 10 credits. Price, review score and location come back on the search result.*

> Take the top result and pull its full detail: facilities, house rules, the room options, and the category ratings.

*One call, 10 credits. Those live on the property page, which the details tool reads by URL and dates.*

> Find four-star hotels in Paris with free cancellation near the center, and list price and review score.

*One call, 10 credits. Star rating, cancellation policy and distance are filters on the one request.*

> Compare the cheapest stay in Paris against Rome for the same dates.

*Two calls, 20 credits, one search per city.*

The property tool needs the same dates and guest counts as the search, because availability and price depend on the window. A search to shortlist plus a detail call on three properties is one search and three property calls.

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_booking_place_getBookingPlaceDetails` | The property identity (hotelId, title, address, coordinates), policies (free cancellation, no prepayment, child/pet stays), price, rating and review summary, photos, and…. 10 credits a call |
| `hasdata_booking_search_getBookingSearchResults` | Each hotel's `hotelId`, title and Booking URL, location info (city, address, coordinates, distance to center / nearest beach), policies (free cancellation, no…. 10 credits a call |

Two tools, read-only. Samples below are trimmed from real calls, and prices move constantly. Read them as shapes. Each tool name links to its endpoint reference, which carries the full field list.

The samples are the payload, not the whole response. A `tools/call` result carries one text block, and that text is itself JSON holding `url`, `status`, `text` and `json`, with the scraped data under `json`. From a raw JSON-RPC response the path is `result.content[0].text`, parsed, then `.json`. A chat client unwraps that for you and code talking to the endpoint directly does not.

### Get Booking.com search results

[`hasdata_booking_search_getBookingSearchResults`](https://docs.hasdata.com/apis/booking/search?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp)

A page of stays by destination and dates.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `keyword` | string | yes | Destination, such as `Paris` or a specific property name |
| `checkInDate` / `checkOutDate` | string | yes | `YYYY-MM-DD`, check-in in the future and before check-out |
| `rooms` / `adults` / `children` | number | yes | Guest composition. Pass `children: 0` when there are none |
| `childrenAges` | string | | Comma-separated ages, required when `children > 0` |
| `sort` | string | | `priceLowestFirst`, `ratingHighToLow`, `bestReviewedAndLowestPrice`, `distanceFromDowntown` and more |
| `propertyType__` / `rating__` / `reviewScore__` | array | | Property type, star rating, and guest-score buckets |
| `facilities__` / `roomFacilities__` / `reservationPolicy__` | array | | Facility, in-room and cancellation filters |
| `price_min_` / `price_max_` | number | | Total-stay price band |
| `page` | number | | About 25 results per page, `2` for the next page |

The reference documents the full filter set, including distance, meals, accessibility, bed preference and travel group.

Returns `searchInformation`, a `results` array, and `pagination` with `page`, `totalResults` and `totalPages`. Each result carries `hotelId`, `title`, `url`, the offered `room` and `bedTypes`, a `location` object, a `policies` object, a `price` object, the star `rating`, a `reviews` object with `score`, `count` and a text label, and a `photo`.

> The discount field in `price` is spelled `dicsount` (`dicsountRaw` and `dicsountParsed`), which mirrors the upstream key. Read that spelling, not `discount`. Also note `rating` is the official star rating while `reviews.score` is the guest score out of 10, two different numbers.

```json
{
  "hotelId": 50724,
  "title": "Hôtel du Jardin des Plantes",
  "url": "https://www.booking.com/hotel/fr/timjardindesplantes.html",
  "room": "Comfort Double Room",
  "location": { "city": "Paris", "address": "5 rue Linné", "mainDistance": "0.9 miles from downtown", "centrallyLocated": true },
  "policies": { "freeCancellation": true, "noPrepayment": true },
  "price": { "pricePerStayParsed": 451.36, "priceBeforeDiscountParsed": 885.03, "dicsountParsed": 433.66, "currency": "USD" },
  "rating": 3,
  "reviews": { "score": 7.5, "count": 1721, "text": "Good" }
}
```

### Get Booking.com property details

[`hasdata_booking_place_getBookingPlaceDetails`](https://docs.hasdata.com/apis/booking/place?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp)

One property in full, by its URL and the stay window.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `url` | string | yes | A Booking.com property URL, the `url` field from a search result |
| `checkInDate` / `checkOutDate` | string | yes | `YYYY-MM-DD`, the window to price and check availability for |
| `rooms` / `adults` / `children` | number | yes | Guest composition, same meaning as the search tool |
| `childrenAges` | string | | Comma-separated ages, required when `children > 0` |

Returns the page as sections rather than one flat object: `overview` (`id`, `title`, `propertyType`, a structured `address`, a `description`, `highlights`, `mostPopularFacilities` and `photos`), `bookingDetails` (the window and currency the prices reflect), a `rooms` array of the available suites each with `name`, `beds`, `facilities` and priced `variants`, a `facilities` list, `houseRules`, a `ratings` array of category scores, `reviews`, and `questionsAndAnswers`.

```json
{
  "overview": {
    "id": "50724",
    "title": "Hôtel du Jardin des Plantes",
    "propertyType": "HOTEL",
    "address": { "country": "France", "zipcode": "75005" },
    "mostPopularFacilities": ["Non-smoking rooms", "Free Wifi", "24-hour front desk"]
  },
  "bookingDetails": { "checkIn": "2026-09-15", "checkOut": "2026-09-18", "adults": 2, "rooms": 1, "currency": "USD" },
  "ratings": [
    { "label": "Average", "value": 7.5, "votes": 1721 },
    { "label": "Cleanliness", "value": 7.8 }
  ]
}
```

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text. The agent reads a message where you might expect a status line.

**A wrong key surfaces as tool output, not as a failed connection.** `tools/list` accepts any non-empty key and returns both tools, so the client completes its handshake and shows green. The first tool call then comes back with `isError: true` and the text `HasData API error: 401 Unauthorized`. Watch for that string, because nothing earlier in the flow reports the problem.

**A missing key is the one real HTTP error.** Authorization runs before any tool, and the connection itself fails with 401. CORS headers are present, and a browser client reads the status and not an opaque network failure.

**An argument that breaks a tool's schema is rejected before it becomes a scrape.** The server answers with `isError: true` and the text `MCP error -32602: Input validation error`, naming the offending field. A `children` count without matching `childrenAges`, or a check-out on or before check-in, is caught here.

**A search with no availability returns a successful result with an empty `results` array**, not an error. A destination and window with nothing open still comes back with `requestMetadata.status` set to `ok`. Test for the array length before you iterate.

**A property URL that no longer resolves returns 400** with `requestMetadata.status` set to `error`.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Each Booking.com tool costs **10 credits per successful call**. Response size does not change the price. A search page of 25 stays costs the same as one with two.

The free tier is **1,000 credits every month with no card**, which is 100 Booking.com calls. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$59 a month** for 200,000 credits, which is 20,000 calls. The unit price falls with volume, from **$2.95 per 1,000 calls** on the entry plan to **$1.19** on Basic and **$0.83** across the Growth tiers. Current figures live on the [pricing page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp).

Your plan also sets concurrency. The free tier allows 1 request at a time, Startup 5, Basic 15, and the Growth tiers run from 50 to 500. Handle the overflow case defensively in anything unattended.

A request that comes back non-200 is not billed. A successful call that finds nothing is still a call.

## Tool selection

The `apis` query parameter decides which tools your agent sees. Fewer tools means less context spent on tool definitions, and fewer chances for the model to reach for the wrong one.

```
?apis=booking                    the two tools in this repo
?apis=booking,airbnb             add Airbnb stays
?apis=booking,google_travel      add Google Hotels and Flights
```

The parameter takes provider names like `booking` and individual API names like `booking_search`. Misspelled names are ignored. If every name is wrong the request fails with 400, and the body lists both what it did not recognise and every valid value. Drop the parameter and the same endpoint exposes all 57 HasData tools.

## How it compares

Booking.com's own programs, the Demand API and the affiliate partner network, are for approved partners who send bookings and earn commission, not a self-serve way to read the public market. For searching stays and reading arbitrary properties, scraping the public pages is the route, and this server does that behind a stable schema.

| | Booking.com partner programs | This server |
| :--- | :--- | :--- |
| Purpose | Send bookings as an approved affiliate | Read the public market |
| Access | Partner approval | One key and one URL |
| Search across the market | Within partner terms | Yes, with rich filters |
| Setup | Business onboarding | None |
| Output | Partner feeds | Structured JSON, price and score pre-parsed |

**What this server does not do.** No booking, no payment, no partner commission, no account data. It reads what a signed-out visitor can see on Booking.com.

## FAQ

### Is there an official Booking.com MCP server?

Booking.com does not publish one. This one is maintained by HasData and reads public Booking.com pages.

### What is a Booking.com MCP server?

A server that exposes Booking.com data as tools an AI client can call. The client sends a tool call over the Model Context Protocol, the server fetches the data and returns structured JSON, and the model works with the result. This one exposes two tools and runs remotely.

### Do I need a Booking.com account or partner approval?

No. The only credential is your HasData key. There is no partner onboarding, because the tools read public Booking.com pages.

### Why does the property tool need dates?

Because availability, room options and price all depend on the stay window. Pass the same `checkInDate`, `checkOutDate` and guest counts you searched with, and the detail reflects that window.

### What is the difference between rating and review score?

`rating` is the official star rating of the property. `reviews.score` is the guest review score out of 10. A three-star hotel can carry a 9.0 guest score, so read the one you mean.

### Can I use this together with other HasData APIs?

Yes. The `apis` parameter takes a list, and `?apis=booking,airbnb` gives your agent Booking.com plus Airbnb. [Drop the parameter](#tool-selection) and you get everything.

### Is HasData affiliated with Booking.com?

No. HasData is an independent service and is not affiliated with, endorsed by, or sponsored by Booking.com. Booking.com is a trademark of its respective owner.

### Compliance and personal data

HasData accesses publicly available data only. A platform's terms may restrict automated access, and you are responsible for your own compliance. Where the data you collect includes personal information, make sure you have a lawful basis for it under GDPR, CCPA or the equivalent rules in your jurisdiction.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [Booking.com Scraper API](https://hasdata.com/apis/booking-api?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| All 57 tools in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| Everything else we scrape | [Booking.com Scraper API and 54 more](https://hasdata.com/apis/?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| Keys and usage | [HasData dashboard](https://app.hasdata.com?utm_source=github&utm_medium=syndication&utm_campaign=booking-mcp) |
| Node launcher on npm | [@hasdata/booking-mcp](https://www.npmjs.com/package/@hasdata/booking-mcp) |
| Python launcher on PyPI | [hasdata-booking-mcp](https://pypi.org/project/hasdata-booking-mcp/) |

## Development

This repository is configuration and documentation for a remote server. There is no build step and nothing to containerize.

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=booking` returns exactly two tools, that every tool still declares its required parameters, that no name changed, and that the key in use is actually accepted. That last check calls a tool for real and costs 10 credits, which is the price of a canary that can fail for the right reason.

```bash
# macOS and Linux
HASDATA_API_KEY=your_key_here npm test

# Windows PowerShell
$env:HASDATA_API_KEY="your_key_here"; npm test
```

The same suite runs in CI on every push and once a week on a schedule, because the upstream tool list can change without anyone touching this repository. A failure means the tool list moved, the key stopped working, or the endpoint was unreachable, and the assertion message says which.

## Contributing

Corrections to the tool tables and the response samples are the most useful contribution, because those are the parts that drift. Include the call you made and the response you got. Pull requests from forks run the suite without a key, and the live checks skip instead of going red.

## License

MIT. See [LICENSE](LICENSE).
