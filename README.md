# get.kantan.sh

One job: serve [kantan](https://github.com/laithalenooz/kantan)'s install script,
so that this works.

```bash
curl -fsSL https://get.kantan.sh | sh
```

## Why a subdomain and not `kantan.sh` itself

The bare domain has to be the website. A single URL cannot honestly answer both
a browser and `curl` — a static host maps one path to one file and never looks
at who asked, so making `curl kantan.sh | sh` work means putting a request
sniffer in front of the site and guessing from the `User-Agent`. A guess in the
one place people pipe bytes into a shell is the wrong trade.

A subdomain has no ambiguity to resolve. Everything here is the script, for
everyone, browser or terminal. It is what rustup does with `sh.rustup.rs`.

## Give curl the scheme

`curl get.kantan.sh | sh` **does not work**, and fails in the worst way: it
prints nothing, exits 0, and installs nothing.

curl assumes `http` when the scheme is left off. The host answers
`308 Permanent Redirect` to `https` with a body of `Redirecting...`. And curl
without `-L` prints that body rather than following it, exiting **0** — so `sh`
receives the word `Redirecting...` and the pipeline looks like it ran. `-f` does
not help, because a 308 is not an error.

Hence `https://` in every example here. With the scheme explicit there is no
redirect to follow, since the `Location` is the URL already being requested.

## The three files

| File | Why |
|---|---|
| `install.sh` | the script, the explicit path: `curl -fsSL https://get.kantan.sh/install.sh \| sh` |
| `index.html` | **byte-identical to `install.sh`** — it is what lets the URL carry no path at all, because a host answering `/` answers with its index |
| `CNAME` | `get.kantan.sh`. Vestigial: this is what GitHub Pages reads, and the domain currently resolves to Vercel (`server: Vercel`, and DNS points at `vercel-dns`). Kept so the repo can be served either way. |

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
