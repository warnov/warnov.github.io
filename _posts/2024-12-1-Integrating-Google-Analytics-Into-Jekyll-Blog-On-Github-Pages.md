---
title: Integrating Google Analytics Into Jekyll Blog On GitHub Pages
date: 2024-12-01 20:35:00 -0500
categories: [GitHub, Pages, Jekyll]
tags: [jekyll, github pages, google analytics, chirpy, customization]
image:
  path: /assets/img/posts/2024-12-1-Integrating-Google-Analytics-Into-Jekyll.webp
---

### Integrating Google Analytics Into Jekyll Blog On GitHub Pages

Deploying a blog with the static CMS **Jekyll** (using the **Chirpy theme** and hosting it via **GitHub Pages**) can be a rewarding but challenging experience, especially when it comes to adding useful features like Google Analytics. If you're finding it difficult to get accurate documentation for this setup, you're not alone. In this guide, I’ll walk you through my journey of integrating Google Analytics into my Chirpy-themed blog, with a focus on some key challenges and how to overcome them.

#### Step 1: Setting Up Google Analytics

First things first: you'll need to set up Google Analytics to generate the tracking code. Follow these steps:

1. **Create a Google Analytics account** (if you haven't already).
2. **Add a new property** for your site. Make sure you specify the correct details like property name, reporting time zone, and currency.
3. **Get the Measurement ID**. This is a string like `G-XXXXXXXXXX`, which is key to integrating Google Analytics with your site.

#### Step 2: Adding Google Analytics to the Chirpy Theme

To integrate Google Analytics, you typically add the tracking ID to your site's `_config.yml` file. However, I found that the default instructions didn’t apply directly for my setup, since some parameters had slightly different names. Here’s how I got it to work:

1. Open the `_config.yml` file in your project.
2. Locate the Google Analytics configuration section. It might not be clearly marked, so you may need to search for the keyword `analytics`.
3. Add the following configuration:
   ```yaml
   google_analytics:
     id: "G-XXXXXXXXXX"  # Replace this with your actual Measurement ID
   ```
   It’s important to note that the structure here can vary depending on which version of the theme you’re using. In my case, it was `google_analytics:` rather than the previously documented `ga:`.

#### Step 3: Challenges with the Chirpy Theme Installation

When you initially clone the Chirpy theme or the Jekyll static CMS, it does not come with all the sources available directly in the repository. Instead, it only downloads a minimal set of sources, as the theme is configured to make references to the larger and complete package that gets installed when you build the site. During that build process, the local configuration of the site is checked, and the full package is downloaded as a RubyGem and installed into the default RubyGems folder of your system. Most of the pieces are referenced from that centralized location during your build process.

This setup is not clearly explained in most of the documentation, and the examples available for integrating Google Analytics typically instruct you to add the tracking script into every page of your site by editing the `_includes/head.html` file. However, if you are new to the Chirpy theme and the Jekyll static CMS, you might be frustrated to find that there is no `_includes` folder in your current project structure, and hence no `head.html` component where you could add the script.

After a lot of prompting and troubleshooting, I discovered a very useful command:

```sh
$ bundle show jekyll-theme-chirpy
```

This command returns the exact folder in your system where the full package (that your build is based on) is downloaded. Upon visiting that folder, I found that the `_includes` folder was present, and within it, there were many components, including `head.html`. I inspected `head.html` and found no Google Analytics section, so I decided to add the script there. The following code snippet is what I added:

```html
{% if jekyll.environment == 'production' and site.google_analytics.id != empty and site.google_analytics.id %}
  <link rel="preconnect" href="https://www.google-analytics.com" crossorigin="use-credentials">
  <link rel="dns-prefetch" href="https://www.google-analytics.com">

  <link rel="preconnect" href="https://www.googletagmanager.com" crossorigin="anonymous">
  <link rel="dns-prefetch" href="https://www.googletagmanager.com">

  <script async src="https://www.googletagmanager.com/gtag/js?id={{ site.google_analytics.id }}"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', '{{ site.google_analytics.id }}');
  </script>
{% endif %}
```

I slightly modified the script that Google provides, such as replacing the hardcoded Analytics code with the variable that was included in `_config.yml`. I also added a condition to ensure the script only runs in production, so my own visits during testing wouldn’t affect the statistics. Additionally, I included some `<link>` tags to preconnect to Google services, which helps boost performance.

#### Step 4: Creating a Local `_includes` Folder for Customization

Since modifying the base theme files directly could interfere with other sites using the same theme (particularly if they do not use Google Analytics), I decided to create a `_includes` folder in my local repository structure. In this folder, I created a `head.html` file and copied all the contents from the original `head.html` file located in the centralized RubyGems folder. I then reverted the original `head.html` file back to its initial state, without the Google Analytics code.

In the local `head.html` file (within my own repo), I retained the Google Analytics script, as shown above. This allowed my customizations to work correctly without affecting any other projects that used the Chirpy theme.

#### Lessons Learned and Additional Customizations

This experience taught me an important lesson about customizing other parts of my blog that I previously couldn't modify because I wasn’t able to locate the relevant components. Many of these components, like the footer and others, were present in the centralized location that was installed when I initially built the site. By identifying them, I could recreate these components in my own repository and make the desired changes. Everything began working smoothly.

I hope these instructions help you work with this awesome static CMS, as it provides a very practical way to publish articles and share your thoughts without needing a server. And what could be better than having your content shared via GitHub Pages, which supports these kinds of static websites? Happy coding!
