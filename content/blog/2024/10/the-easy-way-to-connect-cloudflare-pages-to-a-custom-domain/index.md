---
title: "the easy way to connect cloudflare pages to a custom domain"
date: "2024-10-10"
categories:
  - ""
tags:
  - "adventures-in-software-consulting"
description: "A short tutorial on how to connect cloudflare pages to an external custom domain."
---

Recently I decided to migrate both my technical blog and business website away from WordPress to CloudFlare Pages. For the most part it was a good experience, but I ran into issues trying to connect a custom domain to CloudFlare. I wanted to write a tutorial about this to prevent others from going through the same issue. This guide assumes that you already have a site set up and deployed with CloudFlare Pages and now just need to add a domain.The step-by-step guide is below.

### Connecting CloudFlare Pages to a Custom Domain

1. Go to [your dashboard](https://dash.cloudflare.com).
2. In the left-hand sidebar, select "Workers & Pages" from the drop-down menu.
3. You should now be on the Workers & Pages Overview page. From there, select the project to which you'd like to add a custom domain.
4. Now you should see a page for the project you'd like to deploy using a custom domain. At the top of the page it'll say "Workers & Pages/" followed by the name of your project. If you integrated the site with Github, this will be the name of the repository.
5. Beneath the page heading will be four subheadings: "Deployments", "Functions metrics", "Custom domains", "Integrations", and "Settings." Select "Custom Domains."
6. On this page, you will see a button that says "Set up a Custom Domain." Click the button.
7. From there, it'll ask you to input the name of the domain you own. You can do this and then hit enter.
8. Then you'll land on a page to configure the DNS records appropriately.

### Pitfalls

Here's what you _shouldn't'_ do:

It is possible to add a custom domain from the dashboard - however, this will leave you without the appropriate DNS records, so your site will still not work. If you do it this way, you'll end up with a custom domain that you can see from the dashboard overview page. If you click on the domain name, you'll be taken to a page for that custom domain. At the right hand side, you'll see an option for DNS > DNS Records. If you click "DNS Records," you'll be taken to a page to manually edit the DNS Records. The site will ask you to submit A, AAAA, or CNAME records to finish site setup, but you likely won't know the IP Address for the A or AAAA records or the target for the CNAME records. (The target is the same url as your deployed CloudFlare Pages site - ending in `pages.dev` - but without the `https://` part.) If you set this up to point at the IP Address for anything else from CloudFlare, you'll see an "Error 1000" message when you navigate to your custom domain url.

### Conclusion

And now you have your custom domain connected to CloudFlare Pages!
