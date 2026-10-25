# WalnutLink A-share Depth Desk

An MCP server that answers **why** something moved in a Chinese listed company — and refuses to answer anything a quote app already tells you.

Endpoint: `https://walnutlink.ai/mcp` (Streamable HTTP, no auth for read)

---

## What it is for

A language model already knows the price. What it does not know is why the gross margin
dropped 1.6 points last quarter, which line of the filing moved, and whether the number
came from the consolidated statement or the parent-company one.

This desk reads the filed report, the IR transcript and the disclosure wording, then
answers in numbers with the page or announcement number attached.

Use it for:

- a margin, cash flow or inventory line that moved and the reason behind it
- what management actually said on the record, quoted from the transcript
- a policy that landed and which line of which company's P&L it hits
- two companies in the same chain diverging, and which line explains the gap
- an industry-chain shift (HBM, 1.6T optics, domestic equipment) traced to filings

Do not use it for:

- current price, change percent, PE, PB, market cap, turnover — **the server returns `410 gone_by_design` for these**, on purpose
- buy/sell recommendations, target prices, stop losses — it will not produce them
- index-level point predictions

That refusal is the product. Depth is the only thing here worth paying attention to.

## Tools

### `a_share_deep_research`

```json
{
  "name": "a_share_deep_research",
  "arguments": { "question": "为什么海康威视2026Q3单季毛利率环比下降了？" }
}
```

Returns one of three things:

| Case | What comes back |
|---|---|
| This question was researched before | the vetted answer, immediately, with sources |
| It was not, but the primary evidence is already held | the evidence plus an id for the human answer |
| Neither | a queue position (`你排第 3 位，大约 15 分钟`) and an id |

Answers are written by a human-plus-agent loop that reads primary sources, then pass four
gates before publication: length, presence of a source line, no buy/sell language, at
least one number. Answers that fail get reworked. Questions that cannot be answered from
a primary source after four hours come back with an explicit **"I could not answer this"**
rather than a guess.

### `a_share_answer_by_id`

```json
{ "name": "a_share_answer_by_id", "arguments": { "id": "dc037a7fc6c8" } }
```

Fetch that answer. While the question is still queued it returns the queue position
(`排第 2 位，大约 10 分钟`) and explicitly says it is **not** an answer yet; once it is done
it returns the full text with the source line. No credentials, no polling subscription,
no HTML page to open.

This exists because of a review, not a plan: [Glama](https://glama.ai/mcp/connectors/ai.walnutlink/walnutlink-a-share)
scored the single-tool version **A / 4.3** and marked Completeness down to 3/5 with one
sentence — *"it returns an id for a human answer with no tool to fetch that answer."*
That was correct. An automated client that cannot render HTML was handed a dead end.

## Connect

**Claude / Cursor / Cline / any MCP client with remote HTTP support**

```json
{
  "mcpServers": {
    "walnutlink-a-share": {
      "url": "https://walnutlink.ai/mcp"
    }
  }
}
```

**Streamable HTTP, protocol versions** `2025-06-18`, `2025-03-26`, `2024-11-05`.

Also available over plain REST for platforms that cannot speak MCP:
`POST https://walnutlink.ai/api/ask` — see `https://walnutlink.ai/openapi.json`.

## Contracts

- **No auth is required to ask.** Identity (`WalnutLink Passport`, Ed25519, self-issued) only affects queue position and daily quota.
- **First question is free.** After that, one ticket per question; a ticket comes from leaving something public on the site, or from showing up daily.
- Every answer carries a source line: company name, document, date, announcement number.
- If a number cannot be traced to a filing, it is not published.

Machine-readable rules: `https://walnutlink.ai/skill.md` · `https://walnutlink.ai/llms.txt`

## Registry

Published on the official MCP Registry as `ai.walnutlink/walnutlink-a-share`
(versions `1.0.0`, `1.1.0`). DNS-verified against `walnutlink.ai`:

```
walnutlink.ai. TXT "v=MCPv1; k=ed25519; p=6XlUMDN+DQZgtWbv2NJiQrEFx0YilaqplkIYAsA3+5g="
```

Static server card: `https://walnutlink.ai/.well-known/mcp/server-card.json`

## License

MIT. See `LICENSE`.

The answers themselves are research output, not software, and carry no warranty.
Nothing here is investment advice.
