# The Atelier

A home base for painting with Frank. Mix Van Gogh, Leonardo, and Bosch in the **Studio**, hang Frank's pieces next to hand-made ones in the **Gallery**, talk it over in **Talk**, follow training in the **Lab**, and keep the **Plan** moving.

It's one static file (`index.html`) with no build step and no secrets.

## Put it on GitHub Pages

1. Create a repo, e.g. `atelier`, and add `index.html` (plus these docs) to the root.
2. Go to **Settings → Pages → Build and deployment**, choose *Deploy from a branch*, then `main` / `/ (root)`.
3. About a minute later it's live at `https://<github-user>.github.io/atelier/`.

Pages is free for public repos. A private repo needs GitHub Pro for Pages, and the site itself is public either way. That's fine here: the page holds no data, and everything real lives behind Frank's studio codes.

## Demo mode

With no Frank address set, everything works on its own. Frank is simulated, the art is abstract placeholder sketches, and it's all saved only on that device. It's good for trying the layout today. Reset it from Settings.

## Connecting Frank

The live site is https://trippmorgan.github.io/atelier/. `atelier-api` runs on SuperServer (the hub), reachable only over the family tailnet at `https://tripp-super-server.tail9279ff.ts.net:8443`; Voldemort does the GPU work behind it over SSH (see `FRANK_HANDOFF.md`, whose single-machine notes are superseded by that split). On first visit the site asks for a studio code straight away (the server then says who you are; no name picking). If the address is missing, each person opens **Settings** (the status pill, top right) and enters:

- **Frank's address:** `https://tripp-super-server.tail9279ff.ts.net:8443`
- **Studio code:** their own code (Tripp hands these out privately)

Shortcut: https://trippmorgan.github.io/atelier/?api=https://tripp-super-server.tail9279ff.ts.net:8443 pre-fills the address, so they only type their code. The device has to be on the tailnet first.

## Files

- `index.html`: the app. To change people, painters, or recipes, edit `PEOPLE`, `ARTISTS`, or `PRESETS` near the top of the script.
- `API.md`: the contract Frank implements.
- `FRANK_HANDOFF.md`: the build prompt to give Frank.

The same page is also served by the API itself at https://tripp-super-server.tail9279ff.ts.net:8443/ (tailnet only, same origin, so no CORS). A local copy of index.html (file://, VS Code Live Server on :5500, etc.) is refused by the API's origin check; use one of the two live addresses instead.
