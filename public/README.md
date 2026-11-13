# BUBA GLOBAL INTEGRATED COMPANY: Website + Private Price Manager

A complete Netlify project: public website, live scrap price list, and a private admin page
where management signs in and changes prices. Prices are stored in **Netlify Blobs** and
every change is checked on the **server** (Netlify Functions). There is no password in any
browser file.

```
public/               everything visitors can see (this is the only folder published)
  index.html          public website
  admin.html          private price manager  (/admin also works)
  css/  js/           styles and scripts
  assets/             logo, favicon, images (.webp + .jpg)
netlify/functions/    prices.mjs, admin-login.mjs, admin-session.mjs, lib/shared.mjs
netlify.toml          Netlify settings + security headers
package.json          one dependency: @netlify/blobs
scripts/make_images.py  (optional) regenerates the placeholder artwork
```

---

## 1. Deploy (about 10 minutes)

> **Important:** Netlify's plain drag-and-drop upload does not reliably publish Functions or
> install their dependencies. Use one of the two methods below so the price manager works.

### Option A: GitHub (recommended, easiest to update later)
1. Create a free account at github.com and make a **new private repository**.
2. Unzip this project and upload **all its files** to the repository (GitHub: *Add file > Upload files*).
   Make sure `netlify.toml`, `package.json`, the `public` folder and the `netlify` folder are at the top level.
3. In Netlify: **Add new site > Import an existing project > GitHub**, choose the repository.
4. Leave the build command empty. Publish directory: `public` (already set in `netlify.toml`). Click **Deploy**.
5. Do **step 2 below** (environment variables), then **Deploy > Trigger deploy > Deploy site**.

### Option B: Netlify CLI (from a computer with Node.js 18+)
```bash
npm install
npm install -g netlify-cli
netlify login
netlify init        # or: netlify link  (if the site already exists)
netlify deploy --prod
```
Do step 2 before the final `netlify deploy --prod`, or redeploy afterwards.

---

## 2. Set the private admin password (required)

In Netlify open your site: **Site configuration > Environment variables > Add a variable**.
Add these two (scope: *All scopes*, tick *Contains secret values* if shown):

| Variable | What to put |
|---|---|
| `ADMIN_PASSWORD` | The password you will type to sign in. At least 10 characters. Use a long one nobody can guess. |
| `SESSION_SECRET` | A random string of **at least 32 characters**. It signs the login cookie. |

Generate a good `SESSION_SECRET` with either:
- Computer: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`
- Phone/any: type about 40 random letters and numbers, or use a password manager's generator.

Then **redeploy** (*Deploys > Trigger deploy > Deploy site*). Environment variables only apply to new deploys.

Until both are set, `/admin` shows a "Setup needed" message and nobody can sign in.

**Changing the password later:** edit `ADMIN_PASSWORD`, redeploy. Everyone is signed out automatically.

---

## 3. Using the price manager

1. Open `https://YOUR-SITE.netlify.app/admin` and sign in.
2. Edit a price, name or description directly in the list.
3. **Add a material** with the form at the bottom (name, price, optional description).
4. Remove a material with the **✕** button. Reorder with the arrows.
5. Press **Save changes**. The public website shows the new prices right away.
   Open pages also refresh on their own about every 45 seconds, and when someone returns to the tab.

Before the first save, the website shows the starting prices from the brief
(Mixed Iron ₦500, Special Iron ₦520, Copper ₦14,000, Brass ₦6,000, Aluminum ₦2,200, Can ₦1,500, Zing ₦450).
The material cards, hero price board, ticker, price list and payout estimator all read from the same saved list.

### How it is secured
- Password lives only in Netlify environment variables, never in the code sent to browsers.
- Sign-in sets an `HttpOnly`, `Secure`, `SameSite=Strict` cookie signed on the server; it expires after 8 hours.
- Only a signed-in session can change prices. Anyone can read them.
- 5 wrong passwords from one address locks sign-in for 15 minutes.
- Changing prices also requires a custom request header and same-site origin (protection against forged requests).
- All price input is validated on the server again before saving.

---

## 4. Customise

- **Real photos:** replace files in `public/assets/images/` keeping the same names (`hero`, `about`, `banner`, `mat-copper`, ...) in both `.webp` and `.jpg`. Hero about 1920x1080, material photos 720x540. Free tools like squoosh.app make small `.webp` files.
  The included pictures are generated placeholder artwork, not photographs of your yard.
- **Social media:** in `public/index.html` find the four `data-placeholder` links in the footer, delete `data-placeholder`, and put your real page links in `href`.
- **New material pictures:** a new material uses a generic metal image unless its name contains iron, copper, brass, aluminum, can, zinc/zing.
- **Phone numbers or email:** search `public/index.html` for `2348051639855`, `2348035899834`, and `ibnabubakar80@gmail.com`.
- **Custom domain:** Netlify > Domain management > Add a domain.

## 5. Test on your own computer (optional)
```bash
npm install
npm install -g netlify-cli
cp .env.example .env     # then edit the two values
netlify dev              # opens http://localhost:8888
```

## 6. Troubleshooting
- **Prices on the site never change:** you probably deployed without Functions. Use Option A or B. In Netlify, *Logs > Functions* should list `prices`, `admin-login`, `admin-session`.
- **"Setup needed" on /admin:** variables missing or too short, or you did not redeploy after adding them.
- **"Wrong password":** it is case-sensitive; check `ADMIN_PASSWORD` has no extra spaces.
- **Google Map blank:** the map needs internet access and is not blocked by an ad blocker.
