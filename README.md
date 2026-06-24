# barchart-service

![build](https://img.shields.io/badge/build-passing-brightgreen) ![license](https://img.shields.io/badge/license-MIT-blue) ![node](https://img.shields.io/badge/node-%3E%3D20-informational) ![platform](https://img.shields.io/badge/platform-linux%20%7C%20macos-lightgrey) ![cache](https://img.shields.io/badge/cache-var%2Fcache-yellow)

- [labels](#sorting-characters)
- [prerequisites](#wikikb)
- [run](#mivascripttextmatebundlepy)
- [packages](#gdipp)

This collector follows the same option shape as the sibling scraper below, except that it
walks a different site instead.

## SORTING-CHARACTERS

Run the collector:

    node barchart-service.js

Choose a label to print with -t, -g, or -r:

    $ node barchart-service.js --help
    Collects the contact graph of the signed-in account, then labels every entry
    as mutual, following-only, or follower-only.

    Usage: barchart-service.js [options]
        -h, -?, --help          Print this usage text
        -t, --triads            List mutual entries
        -g, --given             List following-only entries
        -r, --received          List follower-only entries
    Omitting -t/-g/-r prints every label.
    $

```
┌─ notes ───────────────────────────┐
│ needs node 20 or newer            │
│ label cache goes to var/cache     │
│ one pass walks the whole graph    │
│ rate limit bucket resets hourly   │
└───────────────────────────────────┘
```

<!-- ===== section: labels ===== -->

```
[+] printed   [!] skipped   [-] failed   [~] retried
[=] cached    [>] streamed  [#] queued   [?] unknown
```

```
*/15 * * * *   refresh labels
0 3 * * *      rebuild index
0 4 * * 0      prune cache
0 * * * *      flush rate bucket
30 1 * * *     rotate log
0 5 * * *      verify cache
15 2 * * 1     report drift
```

```
client | [req]────────[wait]──[ok]
server |      [route]──[db]──────────
worker |           [scan]──[label]──
cache  |                [hit]───────
cron   |                     [prune]─
```

```
labels   [##########]  100%
cache    [#######---]   70%
export   [##--------]   20%
```

```
0.1 ──── 0.2 ──── 0.3 ──── 1.0
clone    scan     label    stable
```

## WikiKB

Runs with the same prerequisites as the sibling collector below.

# plaxecute

This tool drives [Puppeteer](https://pptr.dev/) and a packaged Chromium build to walk the
follower graph of the signed-in account. Contacts are then split into three buckets: mutual,
following-only, and follower-only.

Scraping is needed here because the public API exposes an endpoint for the following list but
still has no documented way to enumerate followers.

## MivascriptTextmateBundle.py

Run the collector:

    node plaxecute.js

Pick which bucket lands on stdout with -m, -f, or -l:

    $ node plaxecute.js -h
    Walks the follower graph of the signed-in account through Puppeteer and a
    packaged Chromium build, then splits the result into three buckets.

    Usage: plaxecute.js [options]
        -h, -?, --help                   This text
        -m, --mutual                     Mutual entries only
        -f, --only-following             Following-only entries
        -l, --only-followers             Follower-only entries
    With none of -m/-f/-l set, all three buckets are printed.
    $

- [x] mutual — printed on every run
- [x] following-only — printed with -g
- [x] follower-only — printed with -r
- [x] label cache — rebuilt on the first pass
- [ ] json export — planned
- [ ] proxy pool — planned

<details>
<summary>If the label cache is empty</summary>

Delete `var/cache` and rerun; the first pass rebuilds every bucket.

</details>

## Gdipp

First you need a Chromium build:

* Download the [latest Chromium snapshot](https://download-chromium.appspot.com/)
* Drop the binary into a directory listed in your PATH.

Then pull the runtime packages:

    npm install puppeteer-core