# Uploading to GoDaddy (cPanel) — ARM Transport site

What's in this folder
- index.html — the whole website (HTML, CSS and JavaScript are inside this one file)
- models/ — the four MAN fleet models (.glb, 20 MB) and models/lite/ with lighter copies for phones (11 MB)
- .htaccess — tells the server to send .glb files as model/gltf-binary and to cache them (hidden file: turn on
  "Show Hidden Files" in cPanel File Manager settings to see it)
- README.md, _headers, vercel.json — documentation and config for other hosts; harmless to leave

Libraries (Three.js, GSAP) and fonts are loaded from public CDNs when the page opens, the same way most sites do.
Nothing else has to be installed on the hosting.

Steps
1. cPanel → File Manager → open public_html (or the folder your domain points to).
2. Upload arm-transport-website-man-fleet.zip, right-click it → Extract. Then move the contents of the extracted
   "arm-transport-website-man-fleet" folder up into public_html so that index.html and the models/ folder sit directly in it.
   Delete the zip and the empty folder afterwards.
3. Check permissions: files 644, folders 755 (cPanel's defaults).
4. Open https://your-domain/models/man_tge_van.glb — it should download (about 3.6 MB). If you get 403 or 404, the models folder
   is in the wrong place or not readable.
5. Open https://your-domain/#debug — the grey line bottom-right should list all four models as "ready" after a few
   seconds, then disappear. Scroll through the whole page.

If the site is slow to show the first truck, that is the 5 MB hero model downloading; the built-in truck is shown
until it arrives. The models are cached by the browser after the first visit.

Replacing the previous version: delete the old files in models/ (artic.glb, curtainsider.glb, flatbed.glb, tractor.glb,
van.glb and the same names in models/lite/) once the new index.html is in place; the new page does not use them.
