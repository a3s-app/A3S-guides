# Concorde Drupal - Local Development Setup

A guide to setting up and running the Concorde Drupal project locally.

---

## Prerequisites

| Requirement | Download |
|-------------|----------|
| **Lando** | https://lando.dev |
| **Docker Desktop** | https://docker.com/products/docker-desktop |
| **Pantheon Account** | Access to the Concorde project |
| **Machine Token** | [https://dashboard.pantheon.io/users/#account/tokens/](https://dashboard.pantheon.io/personal-settings/machine-tokens) |

---

## Setup Steps

### 1. Clone the Repository

```bash
git clone <repository-url> concorde-www
cd concorde-www
```

### 2. Start Lando Environment

```bash
lando start
```

Wait for all containers to start. You'll see URLs printed at the end.

### 3. Pull Database from Pantheon

```bash
lando pull --database=ada --files=ada --code=none
```

> **First time?** You'll be prompted for a Pantheon machine token.  
> Generate one at: https://dashboard.pantheon.io/users/#account/tokens/

### 4. Clear Cache & Run Updates

```bash
lando drush cr
lando drush updb -y
lando drush cr
```

---

## Accessing the Site

| Service | URL |
|---------|-----|
| **Main Site** | https://concorde-www.lndo.site/ |
| **Admin Login** | Run `lando drush uli` |

---

## How to Add the Local Dev Banner

Follow these steps to add a visual indicator that confirms your local environment is working:

### Step 1: Open the HTML Template

Open the file:
```
web/themes/custom/concorde_bs/templates/layout/html.html.twig
```

### Step 2: Add the Banner Code

Find the `<body>` tag (around line 72) and add the following code **immediately after** it:

```html
<!-- LOCAL DEV BANNER -->
<div id="local-dev-banner" style="position:fixed;top:0;left:0;width:100%;background:#ff6b35;color:white;text-align:center;padding:12px;font-size:16px;font-weight:bold;z-index:99999;box-shadow:0 2px 10px rgba(0,0,0,0.3);">
  🚀 LOCAL DEV ENVIRONMENT - Dev is working here!
</div>
<script>document.body.style.paddingTop = '50px';</script>
<!-- END LOCAL DEV BANNER -->
```

### Step 3: Clear Cache

```bash
lando drush cr
```

---

## Verify Local Setup is Working

After adding the banner, refresh https://concorde-www.lndo.site/

You should see an **orange banner** at the top of every page:

> 🚀 LOCAL DEV ENVIRONMENT - Dev is working here!

![Local Dev Banner Screenshot](../screenshots/local-dev-banner.png)

**If you see the banner, your local environment is working correctly!**

---

## How to Remove the Banner

To remove the banner (e.g., before committing to production):

1. Open `web/themes/custom/concorde_bs/templates/layout/html.html.twig`
2. Delete the lines between `<!-- LOCAL DEV BANNER -->` and `<!-- END LOCAL DEV BANNER -->`
3. Run `lando drush cr`

---

*Last updated: February 9, 2026*
