# Running and hosting Dream Chamber Remixed

[← Back to the README](../README.md)

The repository has the code only. The stems and album covers are copyrighted and are never
committed. The live site at [dreamchamberremixed.xyz](https://dreamchamberremixed.xyz) is hosted on
Cloudflare (Workers static assets); that is the only place it runs for players.

The app is a static site: `index.html`, two style sheets, the scripts under `js/`, the song list
`stems.json` and two compressed copies of every stem (`stems-ogg/` and `stems-flac/`). There is
no server-side code and no build step for the code itself. Opening `index.html` directly from
disk won't work, because browsers block audio and scripts on `file://` pages.

## Work on the code

You need [Node.js](https://nodejs.org) 18+ and nothing else.

```
npm run dev:live  # http://localhost:8138/
```

This serves your working copy and streams the stems and covers from the live site, so you hear
your changes without having the audio. Nothing is saved to disk. The song list is the
`stems.json` in the repo, which matches the live site's stems.

## With your own stems

You need Node.js 18+, [ffmpeg](https://ffmpeg.org) on your PATH and your own copy of the stems as
WAV files.

```
npm run stems     # converts the WAV stems to Ogg and FLAC and writes stems.json
npm run dev       # http://localhost:8137/
```

`npm run stems` looks for the stems in a `Stems/` folder next to this repo by default. To use
a different folder, run `npm run stems -- --stems <path>`. Files must be named
`Stems/<Song>/<Song> - <Groove|Lead|Texture|All> (<Key> <BPM>).wav`, and each one must be a
seamless loop (normally 128 beats; any whole number of bars works). It writes the Ogg copies to
`stems-ogg/`, the FLAC copies to `stems-flac/` and, in `stems.json`, each stem's loudness, length
and alignment mark, which the app checks every download against. The WAV files themselves are
never served. When you add a new song, also add its album and track code to `CATALOG` in
`tools/build-stems.mjs`; a new album needs a page in `ALBUMS` in `js/catalog.js`.

`npm run dev` (`serve.mjs`) serves the repo folder as the hosted site would. It is for development
only: it listens on this computer alone (`localhost`), so it can't be opened from a phone and
must not be put behind a proxy. To try a change on a phone, upload a preview (see below).

To show real album covers, put your images in an `album_cover` folder next to this repo, named
after the album (for example `Random_Access_Memories.jpeg` or `Starboy.jpg`), and run
`npm run covers`. It makes small 256 px copies in `covers/`, which are deployed with the site. Like
the stems, the covers are copyrighted and stay out of git. Without them, the app uses its own
drawn art.

## Host your own copy

The deploy script uploads the site to Cloudflare Workers static assets. To use your own Cloudflare
account or domain, copy `wrangler.jsonc`, change the Worker's name and the domain in `routes` (or
remove `routes` to use the `workers.dev` address), then:

```
npm install                                        # installs Wrangler, Cloudflare's upload tool
npm run stems                                      # if you haven't yet
npx wrangler login
npm run deploy -- --config my-wrangler.jsonc
npm run deploy -- preview --config my-wrangler.jsonc   # or: a preview address, the live site unchanged
```

`npm run deploy` copies the site and the stems into `dist/` and uploads that folder; it refuses to
run if a stem listed in `stems.json` is missing. `_headers` sets the caching: stems are cached for
good (a stem's file name changes when its key or BPM does), the code is checked again on every
visit. [docs/maintaining.md](maintaining.md) has the full release steps for the live site.

`dist/` is plain files, so any static host that serves HTTPS can carry it. Over plain `http`
(other than `localhost`) browsers leave out features the app uses: the true-peak limiter falls
back to a simpler compressor, and Share can't copy to the clipboard.

Anyone who can open the site can download the stems, so only publish audio you're allowed to share.
