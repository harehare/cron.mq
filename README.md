<h1 align="center">cron.mq</h1>

A cron expression parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parse standard 5-field cron expressions (`minute hour day month weekday`)
- `@` aliases: `@yearly`, `@monthly`, `@weekly`, `@daily`, `@hourly`
- Field types: `*`, `*/step`, `start-end`, `a,b,c`, exact value
- Human-readable description via `cron_describe`

## Installation

```sh
cp cron.mq ~/.local/mq/config/
```

### HTTP Import

```sh
mq -I raw 'import "github.com/harehare/cron.mq" | cron::cron_describe(.)' <<< "*/5 * * * *"
```

## API

| Function | Description |
|---|---|
| `cron_parse(expr)` | Parse a cron expression into a structured dict |
| `cron_describe(expr)` | Return a human-readable description of the expression |

### `cron_parse` output

```
{
  "expression": "0 9 * * 1-5",
  "fields": {
    "minute":  { "type": "exact",  "values": [0] },
    "hour":    { "type": "exact",  "values": [9] },
    "day":     { "type": "any",    "values": [] },
    "month":   { "type": "any",    "values": [] },
    "weekday": { "type": "range",  "start": 1, "end": 5, "values": [] }
  }
}
```

## Example

```sh
# Describe a cron expression
mq -I raw 'import "cron" | cron::cron_describe(.)' <<< "*/5 * * * *"
# => "At every 5 minutes past every hour, on every day of the month, in every month, on every weekday"

mq -I raw 'import "cron" | cron::cron_describe(.)' <<< "0 9 * * 1-5"
# => "At 0 past 9, on every day of the month, in every month, on from 1 to 5"

mq -I raw 'import "cron" | cron::cron_describe(.)' <<< "@daily"
# => "At 0 past 0, on every day of the month, in every month, on every weekday"

# Parse to structured data
mq -I raw 'import "cron" | cron::cron_parse(.) | ."fields"."minute"' <<< "*/15 * * * *"
# => {"type":"step","step":15,"values":[]}

# Process a crontab file — describe each job
mq -I raw '
  import "cron"
  | split(., "\n")
  | filter(fn(l): len(trim(l)) > 0 && not(starts_with(l, "#"));)
  | map(fn(l):
      let parts = split(trim(l), "\\s+")
      | let expr = join(slice(parts, 0, 5), " ")
      | let cmd  = join(slice(parts, 5, len(parts)), " ")
      | {"command": cmd, "schedule": cron::cron_describe(expr)}
    ;)
' crontab.txt
```

## License

MIT
