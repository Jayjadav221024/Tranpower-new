# Frontend — React + Vite

Single React app covering the marketing site, the public blog, and the admin
panel, routed with React Router.

```powershell
npm install
npm run dev        # http://localhost:5173
npm run build      # emits dist/ — the backend serves it in production
npm run preview    # preview the production build
```

The dev server proxies `/api` and `/uploads` to `http://localhost:5000`
(see `vite.config.js`), so the browser sees a single origin and the httpOnly
session cookie works without any CORS handling. Start the backend first.

Point it somewhere else with `VITE_API_TARGET=http://host:port npm run dev`.

---

## Layout

```
public/assets/images/       Product photography, served as static files
src/
  main.jsx                  Entry — Router + global.css
  App.jsx                   Route table (public + admin)

  api/client.js             fetch wrapper + typed endpoint helpers
  context/AuthContext.jsx   Session state, resumed from the cookie on mount
  data/products.js          Catalogue, comparison matrix, calculator profiles
  utils/format.js           Dates, file sizes, reading time

  layouts/
    PublicLayout.jsx        Header + Footer shell
    AdminLayout.jsx         Sidebar shell + toast provider

  data/company.js           Contacts, brands, clients, testimonials, achievements
  data/searchIndex.js       What the site search can find, built from the above

  components/
    common/ScrollToTop.jsx  Scroll reset / hash scrolling on navigation
    common/SiteEffects.jsx  Scroll reveal, count-ups, scroll progress bar
    site/                   SiteHeader · SiteFooter · HeroCarousel ·
                            ProductCatalog · LoadCalculator ·
                            ComparisonTable · Applications · RfqForm ·
                            Achievements · GlobalSearch · HomeTicker
    blog/BlogCard.jsx
    admin/                  RequireAuth · MediaPicker · Toast

  pages/
    HomePage · BlogPage · PostPage · NotFoundPage
    admin/                  LoginPage · PostsPage · PostEditorPage ·
                            MediaPage · InquiriesPage · SettingsPage

  styles/
    global.css              Site design tokens + marketing styles
    enhancements.css        Loaded after global.css: phone-layout fixes, entrance
                            motion, the Achievements section and the search UI
    home-motion.css         Looping and pointer-driven motion: hero tilt, the
                            ticker, card tilt/spotlight, ambient colour
    blog.css                Blog listing + article typography
    admin.css               Admin panel
```

## Notes

- **Auth** — `AuthContext` calls `/api/admin/me` once on mount to resume a valid
  cookie. `RequireAuth` waits for that check before redirecting, so a hard
  refresh inside the panel doesn't bounce a signed-in user to the login screen.
- **Hero carousel** — two stacked `<img>` layers crossfade; the incoming image is
  painted on the hidden layer before opacity flips, and the copy swaps at the
  midpoint of the fade. Auto-advances every 3s, pauses on hover.
- **Intro curtain** — the first-visit loading screen is plain HTML/CSS inlined in
  `index.html`, not a React component: it has to be on screen while the bundle
  is still downloading. It plays once per browser tab (`sessionStorage`
  `splash_shown`), never under `/admin`, and lifts when `App.jsx` calls
  `window.__tpAppMounted()`. To see it again, open the site in a new tab.
  Other pages show a top progress bar while their chunk loads (`.route-fallback`).
- **Site search** — `GlobalSearch.jsx` is mounted twice: as the bar at the top of
  the home page, and as a dialog on every page (header search button, `/`, or
  Ctrl/Cmd+K). Products, pages and cities are matched in the browser against
  `data/searchIndex.js`; blog articles are fetched from `/api/posts?q=`. A new
  product page needs a line in `RANGE` in `searchIndex.js` to be findable.
- **Motion** — `SiteEffects.jsx` tags elements by class name and reveals them on
  scroll; nothing is hidden unless that script is running, and it does nothing
  for visitors who ask for reduced motion. It also tracks the mouse: the card
  under the pointer gets `--mx/--my/--rx/--ry`, the hero gets `--px/--py`, and
  `home-motion.css` turns those into the tilt, the spotlight and the parallax.
  Touch devices get the looping motion only. To tone any of it down, each
  effect is one commented block in `home-motion.css` and can be deleted whole.
- **Post content** is rendered with `dangerouslySetInnerHTML`. It comes from the
  admin editor only — see the trust note in the backend README.
- Styles are plain CSS, imported per route. The admin CSS loads only when an
  admin route mounts.
