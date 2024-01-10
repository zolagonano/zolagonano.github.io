---
layout: homepage
title: Webpage
---

# Hey, I'm Zola Gonano 👋

A geek who loves plants, computers, programming, and cryptography.

{% assign post_count = site.posts | size %}
{% if post_count > 0 %}
## Latest blog posts:

Here are my latest blog posts:
{% for post in site.posts limit: 8 %}
- [{{ post.title }}]({{ post.url | relative_url }})({{ post.date | date_to_string }}) {% endfor %}

If you like to checkout my blog, click [here]({{ site.url }}/blog).
{% endif %}

## Projects:

- [CiEnLi](https://github.com/zolagonano/cienli): A library of historical ciphers implemented in rust.
- [STOG](https://github.com/zolagonano/stog): A static blog generator from markdown files.
- [Qr-Api](https://github.com/zolagonano/qr-api): A simple and fast QRcode encoder/decoder API.
- [Rasswd](https://github.com/zolagonano/rasswd): A simple and fast password generator.
- [QrRu](https://github.com/zolagonano/qrru): A CLI tool to encode and decode qr-codes.
- [torbridge-cli](https://github.com/zolagonano/torbridge-cli): A CLI tool to get Tor Bridges from BridgeDB.
- [Awesome Wizard](https://github.com/zolagonano/awesome-wizard): A list of wizard scripts.
- [Awesome ZeroNet](https://github.com/zolagonano/awesome-zeronet): An Awesome & curated list of ZeroNet implementations, plugins, tools, and zites.

## Contact:

If you have something to tell me you can email me at [zolagonano@protonmail.com](mailto:zolagonano@protonmail.com).

If you're not a ProtonMail user and want to send me an encrypted email, you can encrypt your message with my [PGP public key](/assets/public_key.gpg):

- Fingerprint: `F22E B734 505C 76E5 9AFC 95C4 B4A4 AEFD AFF4 8132`

Also, you can follow me on social media:

- [zolagonano](https://github.com/zolagonano) at [github.com](https://github.com/)
- <a rel="me" href="https://mastodon.online/@znano">znano</a> at [mastodon.online](https://mastodon.online/)

## Donate:

- Monero([QR](/assets/qrcodes/monero.png)): `8AF4Lybz7QwiucdYW2szsgiqTHdBp5kjZSSRm6ddzd5363S6n4jixpkACGMLx5JWZnUR5MnGF7cMoidjppruAvLvMe2ovHZ`
- Tron([QR](/assets/qrcodes/tron.png)): `TUT762nFQQRoXvDe1Z72p3kKH9uY3XZCg9`
- DogeCoin([QR](/assets/qrcodes/dogecoin.png)): `DBEaZAbo5tDk7LuXd7pCkhQMa2h8kBgVbS`
- Bitcoin([QR](/assets/qrcodes/bitcoin.png)): `bc1qdzdlytujxn3l02vdt90xx5pkqlezxpuucs6fmm`
- Litecoin([QR](/assets/qrcodes/litecoin.png)): `ltc1qrex5xq3px5qn9vjplfkmvzf7sweks3r4skxe5k`
- BitcoinCash([QR](/assets/qrcodes/bitcoincash.png)): `qzmxuv82j9n2zlxkylrz882yk9e50wvzz5a6hqy2dc`

Full list is available at [/support](/support).
