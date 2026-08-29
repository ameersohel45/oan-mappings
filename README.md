# OAN mappings

Provider mapping files for the OAN network adapter.

**Testing only for now.** These are fetched and executed by the adapter at
runtime, so this repo moves to the `OpenAgriNet` org with branch protection
before it serves real traffic.

## What a mapping is

A YAML file carrying every action one capability serves, each a
[JSONata](https://jsonata.org) expression:

```yaml
actions:
  select: |
    {
      "lat": _local.lat,
      "lon": _local.lon
    }
```

**Request files are keyed by the action they translate** (`select`); **response
files by the action they produce** (`on_select`). Each file names the Beckn
actions it actually deals in, so the filename carries no meaning.

Adding an action is a new key in each file. No new file, and no registry change.

## Layout

```
<participantId>/request.yaml
<participantId>/response.yaml
```

An action a file does not declare is refused, and the error names the ones it
does serve. A mapping that fails to compile takes down only its own action — a
typo in `confirm` is no reason for `select` to stop being served.

## What a mapping can read

| key | request leg | response leg |
|---|---|---|
| `beckn` | the inbound Beckn payload | the inbound Beckn payload |
| `_local` | values the provider plugin resolved | the same values |
| `response` | — | the provider's raw answer |

`_local` stays in scope on the response leg on purpose. A provider's answer
rarely repeats what it was asked, so values resolved before the call are often
the only source for them in the output — the coordinates of a forecast, say.

## Wiring one up

The adapter finds these through the registry. A `ProviderSchema` row names them:

```json
{
  "bindingKey": "mausamgram|openagrinet:WeatherObservation",
  "requestMapping":  "https://raw.githubusercontent.com/ameersohel45/oan-mappings/main/mausamgram/request.yaml",
  "responseMapping": "https://raw.githubusercontent.com/ameersohel45/oan-mappings/main/mausamgram/response.yaml"
}
```

Adding a provider is a row plus two files. No adapter release.

## Two things to know

**Changes are not instant.** `raw.githubusercontent.com` caches for around five
minutes, and the adapter caches compiled mappings on top of that
(`cacheTTL`, default 1h). Worth remembering before debugging why a fix has not
landed.

**Omit rather than emit empty.** A reading the provider did not take should be
absent from the output, not present and blank — a consumer has to be able to
tell "not recorded" from "zero". See `select.response.yaml` for how.
