# Setting Up a Custom Domain for Your Astro Site

This guide will walk you through the process of setting up a custom domain for your Astro site that's hosted on GitHub Pages.

## Current Setup

Your site is currently configured to be deployed to GitHub Pages with:
- Site URL: `https://fivetwentythree.github.io/real/`
- Base path: `/real/`

## Steps to Set Up a Custom Domain

### 1. Purchase a Domain

First, purchase a domain from a domain registrar like Namecheap, GoDaddy, or Google Domains.

### 2. Configure GitHub Repository Settings

1. Go to your GitHub repository that hosts this site
2. Navigate to **Settings > Pages**
3. Under **Custom domain**, enter your domain name (e.g., `yourdomain.com`)
4. Click **Save**
5. Check the box for **Enforce HTTPS** (recommended for security)

### 3. Update DNS Records at Your Domain Registrar

You have two options:

#### Option A: Using an A Record
Create four A records pointing to GitHub Pages' IP addresses:
```
A @ 185.199.108.153
A @ 185.199.109.153
A @ 185.199.110.153
A @ 185.199.111.153
```

#### Option B: Using a CNAME Record
If you want to use a subdomain (like `www.yourdomain.com`):
```
CNAME www yourusername.github.io
```

### 4. Create a CNAME File

Create a file named `CNAME` (no file extension) in the `public` directory with your domain name as its only content:

```
yourdomain.com
```

This file should be placed in the `public` directory so it gets copied to the root of your built site.

### 5. Update Site Configuration

Modify the `src/site.config.ts` file to use your custom domain:

```typescript
export const siteConfig: SiteConfig = {
  // other config...
  url: "https://yourdomain.com",
};
```

And update the `astro.config.ts` file to remove the base path:

```typescript
export default defineConfig({
  site: siteConfig.url,
  // Remove or comment out the base: "/real/" line
  // base: "/real/",
  // rest of config...
});
```

Also update the menu links in `src/site.config.ts` to remove the `/real/` prefix:

```typescript
export const menuLinks: { path: string; title: string }[] = [
  {
    path: "/",
    title: "Home",
  },
  {
    path: "/about/",
    title: "About",
  },
  // other links...
];
```

### 6. Commit and Push Changes

Commit these changes and push them to your repository. The GitHub Actions workflow will automatically deploy your site with the new configuration.

### 7. Wait for DNS Propagation

DNS changes can take up to 48 hours to propagate, though they often take effect much sooner. Once propagation is complete, your site should be accessible via your custom domain.

## Troubleshooting

- If your site isn't accessible after 48 hours, check your DNS settings and GitHub Pages configuration.
- Ensure the CNAME file is correctly placed in the public directory and contains only your domain name.
- Verify that your GitHub Pages site is being built and deployed correctly by checking the Actions tab in your repository.

## Additional Resources

- [GitHub Pages Custom Domain Documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
- [Astro Deployment Documentation](https://docs.astro.build/en/guides/deploy/)