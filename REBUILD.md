# Rebuilding evanlih.photography from scratch

How to set the whole site up again if anything breaks or gets deleted: the repo, GitHub Pages, Cloudflare and the domain. This file lives at the repo root, so it never shows up on the site. For day-to-day changes (adding photos or tabs), see [README.md](README.md).

Time needed: about 20 minutes of clicking, plus up to a few hours for DNS changes to spread.

## What you need

- A **GitHub** account (`EvanLih`)
- A **Cloudflare** account (free plan is fine)
- The domain **evanlih.photography**, registered somewhere (check the Cloudflare dashboard to see where it's registered and when it renews)
- This repo's code (on GitHub, or the copy in `~/Documents/Github/evanlih.github.io`)

## The big picture

```
you push to main ──► GitHub Actions builds the site with Hugo ──► GitHub Pages hosts it
                                                                        ▲
visitor types evanlih.photography ──► Cloudflare (DNS + proxy) ─────────┘
```

## Step 1. Put the code on GitHub

1. Create a repo named **`evanlih.github.io`** under the `EvanLih` account. It must be public on a free plan.
2. Push this code to its `main` branch.

The build script is already in the repo at `.github/workflows/hugo.yaml`. It's GitHub's sample "Deploy Hugo site to Pages" workflow, with the site address filled in.

## Step 2. Turn on GitHub Pages

1. In the repo, go to **Settings → Pages**: <https://github.com/EvanLih/evanlih.github.io/settings/pages>
2. Under **Build and deployment → Source**, choose **GitHub Actions**.
3. Go to the **Actions** tab and check that the "Deploy Hugo site to Pages" run finishes with a green check. Re-run it if needed: Actions → Deploy Hugo site to Pages → Run workflow.

At this point the site works at `https://evanlih.github.io`.

Docs: [Configuring a publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

## Step 3. Put the domain on Cloudflare

1. In Cloudflare, **Add a domain** → `evanlih.photography` → Free plan.
2. Cloudflare gives you two nameservers. At the domain registrar, replace the domain's nameservers with those two. (Skip this if the domain is already on Cloudflare: the DNS page then shows **"DNS Setup: Full"**.)

## Step 4. Add the DNS records

In Cloudflare: **evanlih.photography → DNS → Records → Add record**. Add these five, with Proxy status on (orange cloud) for all of them:

| Type | Name | Content | Proxy status | TTL |
|---|---|---|---|---|
| A | `evanlih.photography` (enter `@`) | `185.199.108.153` | Proxied | Auto |
| A | `evanlih.photography` (enter `@`) | `185.199.109.153` | Proxied | Auto |
| A | `evanlih.photography` (enter `@`) | `185.199.110.153` | Proxied | Auto |
| A | `evanlih.photography` (enter `@`) | `185.199.111.153` | Proxied | Auto |
| CNAME | `www` | `evanlih.github.io` | Proxied | Auto |

These come from GitHub's docs:
- The four A records: [Configuring an apex domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)
- The www CNAME: [Configuring an apex domain and the www subdomain variant](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain-and-the-www-subdomain-variant)

GitHub can change those four addresses. If the site stops resolving someday, check that page for the current ones.

## Step 5. Tell GitHub Pages about the domain

1. Back in **Settings → Pages → Custom domain**, enter `evanlih.photography` and save.
2. Wait for the DNS check to pass. It can take a while after Step 4.
3. **Enforce HTTPS** may be greyed out. That's expected with the orange cloud on, because Cloudflare supplies the padlock instead of GitHub.

## Step 6. Set Cloudflare's SSL mode

In Cloudflare: **SSL/TLS → Overview**, set the encryption mode to **Full**.

Don't use **Flexible**. It can cause an endless "too many redirects" loop with GitHub Pages. Docs: [ERR_TOO_MANY_REDIRECTS](https://developers.cloudflare.com/ssl/troubleshooting/too-many-redirects/)

## Step 7. Make sure Hugo knows the address

These two places must say `https://evanlih.photography` (they already do in this repo):

- `config/_default/hugo.toml` → `baseURL = "https://evanlih.photography"`
- `.github/workflows/hugo.yaml` → the `hugo` build command's `--baseURL "https://evanlih.photography/"`

If you ever change domains, update both, then push.

## Step 8. Check it works

- [ ] Latest run in the **Actions** tab is green
- [ ] <https://evanlih.photography> loads with the padlock
- [ ] <https://www.evanlih.photography> also loads
- [ ] Every tab loads photos, on desktop and phone

## What the DNS terms mean

Think of DNS as the internet's phone book. People type a name, computers connect to a number (an IP address), and DNS records are your entries in that phone book.

- **Apex vs www:** the apex is the bare name `evanlih.photography`. `www.evanlih.photography` is technically a separate name, so it needs its own record.
- **A record ("Address"):** points a name straight at a number. The four A records are GitHub Pages' servers. There are four so that if one is down, the others answer.
- **CNAME ("Canonical Name"):** points a name at another name instead of a number. `www` points at `evanlih.github.io`, so it keeps working even if GitHub changes its addresses. The apex uses A records because DNS doesn't allow a plain CNAME on a bare domain.
- **Content:** where the record points.
- **Proxied (orange cloud):** visitors connect to Cloudflare, and Cloudflare fetches the page from GitHub and passes it along. That gives you Cloudflare's https padlock and caching, and keeps GitHub's addresses hidden. With "DNS only" (grey cloud), visitors would go straight to GitHub.
- **TTL (Time to Live):** how long other computers remember the answer before asking again. With the proxy on, Cloudflare manages it ("Auto").

## Troubleshooting

| Problem | Likely fix |
|---|---|
| "Too many redirects" | Cloudflare SSL/TLS mode is Flexible. Set it to Full. |
| Site shows a GitHub 404 page | The Pages custom domain was cleared. Re-enter `evanlih.photography` in Settings → Pages. |
| Push didn't update the site | Check the Actions tab for a red X and open the failed step's log. |
| Domain doesn't load at all | Check the five DNS records above, and that the domain hasn't expired at the registrar. |
| Photos missing or layout broken | Theme update issue. See "Update the theme" in README.md. |

More: [Troubleshooting custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages)

## Step 9. Block email spoofing

This domain isn't used for email. These three records tell every mail server "evanlih.photography never sends email, so reject anything claiming to come from it," which stops spammers from faking mail as @evanlih.photography.

Easiest way to add them: Cloudflare's DNS page shows a recommendation about email spoofing. Its link opens the **Email Record Creator**, which fills in all three. Or add them by hand. All three are **DNS only** (grey cloud):

| Type | Name | Content | Proxy status | TTL |
|---|---|---|---|---|
| TXT | `evanlih.photography` (enter `@`) | `v=spf1 -all` | DNS only | Auto |
| TXT | `*._domainkey` | `v=DKIM1; p=` | DNS only | Auto |
| TXT | `_dmarc` | `v=DMARC1; p=reject; sp=reject; adkim=s; aspf=s;` (plus an optional `rua=` reporting address) | DNS only | Auto |

What each one does:
- **SPF** lists the servers allowed to send mail for the domain. `-all` with nothing before it means none.
- **DKIM** normally publishes a key for signing email. The empty key (`p=`) means no message can ever pass.
- **DMARC** says what to do with mail that fails those checks: `p=reject` bounces it, `sp=reject` covers subdomains, and `adkim=s; aspf=s` means strict matching. `rua=` (optional) is where mail servers send summary reports.

These don't affect the website. **If you ever want email at this domain** (like hello@evanlih.photography), replace all three with your email provider's settings first, or mail from the domain will be rejected.

## Reference links

- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) (the DNS records)
- [About custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)
- [Secure and fast GitHub Pages with Cloudflare](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare) (Cloudflare's write-up of this setup; older, so screenshots may differ)
- [Bridget theme](https://github.com/Sped0n/bridget)
- [Hugo](https://gohugo.io)
