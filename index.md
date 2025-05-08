---
layout: homepage
title: Zola Gonano
description: Webpage of a pseudonymous cryptoanarchist
image: /assets/sauropods/sauropod.png
---

# Hey, I'm Zola 👋

Just a pseudonymous cryptoanarchist trying to change what I can for the better—what more could there possibly be?

{% assign post_count = site.posts | size %}
{% if post_count > 0 %}
## Recent blog posts:

I have a blog where I write about technical topics that interest me. Check out my latest blog posts, and you can explore more on my blog at [/blog]({{ site.url }}/blog).

{% for post in site.posts limit: 8 %}
- {{ post.date | date_to_string }}: [{{ post.title }}]({{ post.url | relative_url }}) {% endfor %}

{% endif %}

## Highlighted Projects:

Here are some of my projects; you can check the full list at [/projects]({{ site.url }}/projects).

{% for project in site.data.projects %}{% if project.highlight %}- [{{ project.name }}]({{ project.url}} ): {{ project.description }}
{% endif %}{% endfor %}

## Contact:

If you have something to tell me you can email me at [zolagonano@protonmail.com](mailto:zolagonano@protonmail.com).

If you're not a ProtonMail user and want to send me an encrypted email, you can encrypt your message with my [PGP public key](/assets/public_key.gpg):

- Fingerprint: `F22E B734 505C 76E5 9AFC 95C4 B4A4 AEFD AFF4 8132`

Also, you can follow me on social media:

- [zolagonano](https://github.com/zolagonano) at [github.com](https://github.com/)
- <a rel="me" href="https://mastodon.online/@znano">znano</a> at [mastodon.online](https://mastodon.online/)
- [znanoznano](https://x.com/znanoznano) at [twitter.com](https://x.com/)

## Donate/Support:

 I dedicate my free time to working on Free and Open Source Software (FOSS) and creating free content. Your donations enable me to invest more time in these projects.

Here are some ways you can donate, and a full list is available at [/support]({{ site.url }}/support):

{% for crypto in site.data.crypto_donations %}{% if crypto.highlight %}- **{{ crypto.name }}** _([QR]({{ crypto.qr }}))_: `{{ crypto.address }}`
{% endif %}{% endfor %}

