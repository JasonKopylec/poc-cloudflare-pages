# Cloudflare Pages Setup Guide

A step-by-step guide to deploy your git repository using Cloudflare Pages free tier.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Create a Cloudflare Account](#create-a-cloudflare-account)
3. [Connect Your GitHub Repository](#connect-your-github-repository)
4. [Configure Build Settings](#configure-build-settings)
5. [Deploy Your Site](#deploy-your-site)
6. [Verify Your Deployment](#verify-your-deployment)
7. [Custom Domain (Optional)](#custom-domain-optional)

---

## Prerequisites

Before you start, make sure you have:

- A GitHub account with your repository pushed to it
- A repository containing your static HTML files (like your `index.html`)
- A web browser

> **Note**: This guide assumes you're deploying a static site (HTML, CSS, JavaScript). Cloudflare Pages free tier supports static site generators too (Hugo, Jekyll, etc.).

---

## Create a Cloudflare Account

### Step 1: Visit Cloudflare

1. Go to [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
2. You'll see the Cloudflare sign-up page

### Step 2: Sign Up

You can sign up using:
- **Email and password**: Enter your email, create a strong password, check the terms, and click **"Create account"**
- **Google account**: Click **"Sign up with Google"** and follow the prompts (easiest option)
- **GitHub account**: Click **"Sign up with GitHub"**

### Step 3: Complete Setup

You'll be automatically logged in and taken to your Cloudflare Dashboard. You may be asked about your use case (optional - you can skip this).

---

## Connect Your GitHub Repository

### Step 1: Go to Workers & Pages

1. In your Cloudflare Dashboard, click **"Workers & Pages"** in the left sidebar
2. You should land on the Workers & Pages page

### Step 2: Create Your Application

1. Click **"Create application"**
2. Select **"Pages"**
3. Click **"Connect to Git"**

### Step 3: Authorize GitHub

1. You'll be prompted to authorize Cloudflare to access your GitHub account
2. Click **"Install & Authorize"** or **"Authorize Cloudflare"**
3. You may need to log in to GitHub if you're not already logged in
4. Grant Cloudflare permission to access your repositories

### Step 4: Select Your Repository

1. After authorization, you'll see a list of your GitHub repositories
2. Find and click on your `poc-cloudflare-pages` repository
3. Click **"Begin setup"**

### Step 5: Configure Your Repo Details

On the "Set up builds and deployments" page:

1. **Project name**: This will be your site's subdomain (e.g., `poc-cloudflare-pages.pages.dev`)
   - You can use the default or customize it
   - This name must be unique across Cloudflare Pages
2. **Production branch**: Set this to `main` (the branch Cloudflare will deploy automatically)

---

## Configure Build Settings

### Step 1: Build Configuration

After configuring your project name and production branch, you'll see the build settings page:

- **Framework preset**: Select `None` (since you're deploying static HTML)
  - If you had a build process, you'd select the appropriate framework
- **Build command**: Leave blank (your static files don't need building)
- **Build output directory**: Leave as `/` or empty (your `index.html` is in the root)

### Step 2: Environment Variables (Optional)

You can skip this section for a basic static site.

### Step 3: Root Directory (Optional)

Leave this blank unless your content is in a subdirectory.

### Step 4: Save and Deploy

Once you're done configuring:

1. Click **"Save and Deploy"**
2. Cloudflare will begin building and deploying your site

---

## Deploy Your Site

### Initial Deployment

1. After clicking "Save and Deploy", Cloudflare will:
   - Clone your repository
   - Build your site (if a build command is configured)
   - Deploy it to their CDN

2. You'll see a deployment page showing:
   - Build status (should show "Success")
   - A URL like `https://poc-cloudflare-pages.pages.dev`

3. Wait for the deployment to complete (usually takes 30 seconds to 2 minutes)

### Automatic Deployments (Git-Connected)

Since you've connected to GitHub, automatic deployments are already enabled! Here's how it works:

**Every time you push to `main`:**
1. GitHub notifies Cloudflare of the push
2. Cloudflare automatically pulls your latest code
3. Cloudflare builds and deploys your site
4. Your changes go live at your Pages URL

**To deploy a change:**
1. Make changes to your files locally
2. Commit your changes: `git commit -m "Your message"`
3. Push to main: `git push origin main`
4. Check the Cloudflare dashboard to see the deployment progress

**To view deployment history:**
1. Go to your Cloudflare dashboard → Workers & Pages
2. Click on your project
3. Click the **"Deployments"** tab
4. You'll see all past deployments with their status, timestamp, and commit info

### Preview Deployments

All branches other than `main` get automatic preview deployments:
- Push to any other branch (e.g., `feature-branch`)
- Cloudflare automatically deploys to: `<branch-name>.<project-name>.pages.dev`
- Useful for testing before merging to main
- You can roll back to any previous deployment from the dashboard if needed

---

## Verify Your Deployment

### Step 1: Check Deployment Status

1. In the Cloudflare dashboard, go to **Workers & Pages** → Your Project
2. Click the **"Deployments"** tab
3. You should see your latest deployment with a **"Success"** status
4. It will show your commit message and timestamp

### Step 2: Visit Your Site

1. Once deployment succeeds, click the deployment
2. Copy your site URL (e.g., `https://poc-cloudflare-pages.pages.dev`)
3. Visit the URL in your browser
4. You should see your `index.html` rendered with all content, images, and styling

### Step 3: Test Responsiveness

1. Test your site on different devices/screen sizes
2. Check that YouTube video thumbnails load correctly
3. Verify all links work

### Step 4: Test Auto-Deployment

To confirm automatic deployments are working:

1. Make a small change to `index.html` locally
2. Commit and push: `git add . && git commit -m "test deployment" && git push origin main`
3. Watch the Cloudflare dashboard
4. A new deployment should appear within a few seconds
5. Once it shows "Success", refresh your site in the browser to see your changes live

---

## Custom Domain (Optional)

If you want to use your own domain instead of `*.pages.dev`:

### Step 1: Point Your Domain to Cloudflare

1. You'll need to change your domain's nameservers to Cloudflare's
2. In the Cloudflare dashboard, you can set this up under the domain section

### Step 2: Add Domain to Pages Project

1. In your Pages project settings, go to the **"Custom domains"** tab
2. Click **"Set up a custom domain"**
3. Enter your domain name (e.g., `dhammarato-poc.com`)
4. Follow the DNS configuration steps provided by Cloudflare

### Step 3: Configure DNS Records

Cloudflare will guide you through creating the necessary DNS records to point your domain to your Pages deployment.

---

## Troubleshooting

### Deployment Not Starting

**Problem**: After pushing to GitHub, no new deployment appears in Cloudflare

**Solution**:
1. Make sure you pushed to the correct branch (`main`)
2. Check that your repository is properly connected in Cloudflare
3. Go to **Workers & Pages** → Your Project → **Settings** → **Git Configuration**
4. Verify the correct GitHub repository and branch are selected
5. Try disconnecting and reconnecting your GitHub account

### Build Failed

**Problem**: Deployment shows "Build failed"

**Solution**:
1. Click on the failed deployment to see build logs
2. Look for error messages
3. Common issues for static sites:
   - Incorrect build output directory setting (should be `/` for root)
   - Syntax errors in HTML, CSS, or JavaScript
4. Fix the error, commit, and push again

### Site Not Loading

**Problem**: You get a 404 or blank page

**Solution**:
1. Verify `index.html` is in the root of your repository
2. Go to **Workers & Pages** → Your Project → **Settings** → **Build Configuration**
3. Confirm Build output directory is correct (leave blank or `/` for static sites with files in root)
4. Clear your browser cache (Ctrl+Shift+Delete or Cmd+Shift+Delete)
5. Wait a few moments for the site to load

### Images or Videos Not Loading

**Problem**: Images appear as broken links or YouTube thumbnails don't show

**Solution**:
1. Verify image/video URLs in your HTML are correct
2. Check that all image files are in your repository
3. If using relative paths, make sure they're relative to your HTML file
4. Clear browser cache and hard refresh (Cmd+Shift+R on Mac, Ctrl+Shift+R on Windows)

### DNS Error After First Deploy

**Problem**: You see a DNS error after visiting your site for the first time

**Solution**:
1. DNS can take a few minutes to propagate
2. Wait 5-10 minutes and try again
3. Try opening the site in a private/incognito window
4. Try from a different network or device

---

## Next Steps

Once your site is deployed:

1. **Monitor deployments**: Check the Pages dashboard regularly to ensure deployments succeed
2. **Add custom domain**: Point a real domain to your Pages project (see above)
3. **Enable advanced features**:
   - Custom headers and redirects (in `_headers` and `_redirects` files)
   - Form handling
   - Cron jobs (with Cloudflare Workers)

---

## Free Tier Limits

The Cloudflare Pages free tier includes:

- **Unlimited sites**: Deploy as many projects as you want
- **Unlimited bandwidth**: No data limits
- **Unlimited requests**: All the traffic you need
- **GitHub integration**: Automatic deployments
- **SSL/TLS**: Free HTTPS for all projects
- **50 builds per month**: Enough for regular updates

> **Note**: If you need more than 50 builds per month, consider upgrading or using git squashing to reduce commit frequency.

---

## Additional Resources

- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
- [Cloudflare Pages GitHub Integration](https://developers.cloudflare.com/pages/get-started/guide/#connect-your-git-account)
- [Custom Domains on Pages](https://developers.cloudflare.com/pages/configuration/custom-domains/)
- [Cloudflare Community Forum](https://community.cloudflare.com/)

---

## Summary

You now have a fully functioning, globally distributed static site hosted on Cloudflare Pages! Your site will:

- ✅ Auto-deploy when you push to GitHub
- ✅ Be served from Cloudflare's global CDN
- ✅ Have free HTTPS
- ✅ Get incredible performance
- ✅ Have automatic rollback capability

Enjoy your deployment!
