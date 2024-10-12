---
title: "the easy way to connect cloudflare pages to a custom domain"
date: "2024-10-12"
categories:
  - ""
tags:
  - "tutorials"
description: "A short tutorial on the right way to connect CloudFlare Pages to an external custom domain."
---

Recently I decided to migrate both my technical blog and business website away from WordPress to Eleventy. As part of that process I also decided to deploy my website myself using CloudFlare Pages. I ran into several issues trying to connect a custom domain to CloudFlare, so I decided to write a tutorial about it to save others some time.

Note: This guide assumes that you already have a site set up and deployed with CloudFlare Pages and now just need to add a domain.

### Steps to Connect CloudFlare Pages to a Custom Domain

1. Go to <a href="https://dash.cloudflare.com" target="_blank">your dashboard</a>.
2. In the left-hand sidebar, select "Workers & Pages" from the drop-down menu.
3. You should now be on the Workers & Pages Overview page. From there, select the project to which you'd like to add a custom domain.
4. Now you should see a page for the project you'd like to deploy using a custom domain. At the top of the page it'll say "Workers & Pages/" followed by the name of your project. If you integrated the site with Github, this will be the name of the repository.
5. Beneath the page heading will be four subheadings: "Deployments", "Functions metrics", "Custom domains", "Integrations", and "Settings." Select "Custom Domains."
6. On this page, you will see a button that says "Set up a Custom Domain." Click the button.
7. From there, it'll ask you to input the name of the domain you own. You can do this and then hit enter.
8. You will be asked to migrate your domain to CloudFlare DNS. A new window will pop up and you'll need to input the custom domain once again. From there you'll have the option to let CloudFlare scan for your existing DNS records or to input them manually. For easy site setup you can do the former.
9. On the next screen, you'll be asked to select a plan. You can choose the free tier.
10. Then you'll be asked to review the imported DNS records. If all looks well you can click "continue to activation."
11. You'll be given a list of two name servers. You will need to update your domain registrar to point to these two name servers. This can take up to 48 hours.
12. Once your domain name is active on CloudFlare, you should receive an email alerting you.
13. (optional) To set `www` to redirect to the domain apex, you'll need to <a href="https://developers.cloudflare.com/pages/how-to/www-redirect/" target="_blank">follow these instructions</a>

### Pitfalls

Here's the tricky part - there are two places where you can add a custom domain, but only one option will allow you to configure your page correctly. You can add a custom domain using the steps outlined above _or_ from the dashboard overview page where you may see an option to "add a domain." The steps that you follow will be almost identical, except you won't be given the option to add the appropriate CNAME record. You can input this manually - the name will be your domain name, and the content will be the link to your site using a CloudFlare url that ends in `pages.dev`, minus `https://` or `http://`.

### Conclusion

And now you should be good to go! You've successfully connected your custom domain to CloudFlare Pages and you're free to share your website with the world.

---

Want to be notified when new tutorials like this one are posted? Subscribe below!
