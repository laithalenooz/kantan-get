# get.kantan.sh

One job: serve [kantan](https://github.com/laithalenooz/kantan)'s install script,
so that this works.

```bash
curl get.kantan.sh | sh
```

## Why a subdomain and not `kantan.sh` itself

The bare domain has to be the website. A single URL cannot honestly answer both
a browser and `curl` — a static host maps one path to one file and never looks
at who asked, so making `curl kantan.sh | sh` work means putting a request
sniffer in front of the site and guessing from the `User-Agent`. A guess in the
one place people pipe bytes into a shell is the wrong trade.

A subdomain has no ambiguity to resolve. Everything here is the script, for
everyone, browser or terminal. It is what rustup does with `sh.rustup.rs`.

## The three files

| File | Why |
|---|---|
| `install.sh` | the script, the explicit path: `curl get.kantan.sh/install.sh \| sh` |
| `index.html` | **byte-identical to `install.sh`** — it is what makes the bare `curl get.kantan.sh` work, because a host answering `/` answers with its index |
| `CNAME` | `get.kantan.sh`, which is what GitHub Pages reads |

`index.html` being a shell script is the whole trick, and it is less strange
than it looks: `curl` does not read the content type, and `sh` reads bytes. A
browser does render it as HTML, which is why **the script deliberately contains
no `<`** — otherwise a reader checking it before piping it would be shown a line
with a word quietly missing. Avoiding one character beat configuring a host to
send `text/plain`.

## Keeping it in step

The script's home is [`kantan.sh` in the main repo](https://github.com/laithalenooz/kantan/blob/master/kantan.sh).
This repo is a copy for serving. When it changes there:

```bash
cp ../kantan/kantan.sh install.sh && cp install.sh index.html
```

Both files must stay byte-identical, or the bare URL and the explicit path hand
out different scripts — which is the one difference nobody would think to check.

## Licence

MIT, as kantan is.
