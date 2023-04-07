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
{% for post in site.posts limit: 5 %}
- [{{ post.title }}]({{ post.url | relative_url }})({{ post.date | date_to_string }}) {% endfor %}

If you like to checkout my blog, click [here]({{ site.url }}/blog).
{% endif %}

## Projects:

- [CiEnLi](https://github.com/zolagonano/cienli): A library of historical ciphers implemented in rust.
- [STOG](https://github.com/zolagonano/stog): A static blog generator from markdown files.
- [Qr-Api](https://github.com/zolagonano/qr-api): A simple and fast QRcode encoder/decoder API.
- [Rasswd](https://github.com/zolagonano/rasswd): A simple and fast password generator.
- [QrRu](https://github.com/zolagonano/qrru): A CLI tool to encode and decode qr-codes.
- [Awesome Wizard](https://github.com/zolagonano/awesome-wizard): A list of wizard scripts.
- [Awesome ZeroNet](https://github.com/zolagonano/awesome-zeronet): An Awesome & curated list of ZeroNet implementations, plugins, tools, and zites.

## Contact:

If you have something to tell me you can email me at [zolagonano@protonmail.com](mailto:zolagonano@protonmail.com).

If you're not a ProtonMail user and want to send me an encrypted email, you can encrypt your message with my [PGP public key](/assets/public_key.gpg):

- Fingerprint: `F22E B734 505C 76E5 9AFC 95C4 B4A4 AEFD AFF4 8132`

Also, you can follow me on social media:

- [zolagonano](https://github.com/zolagonano) at [github.com](https://github.com/)
- [znano](https://mastodon.online/@znano) at [mastodon.online](https://mastodon.online/)

## Donate:

- Monero([QR](/assets/qrcodes/monero.png)): `8AF4Lybz7QwiucdYW2szsgiqTHdBp5kjZSSRm6ddzd5363S6n4jixpkACGMLx5JWZnUR5MnGF7cMoidjppruAvLvMe2ovHZ`
- VerusCoin([QR](/assets/qrcodes/veruscoin.png)): `R9V91vQbP75A5H3Nn3RrXnK8zVZyaRBYHG`
- Bitcoin([QR](/assets/qrcodes/bitcoin.png)): `bc1qtya7nc42xff4w8rw6xa9zeqhdk4s3telvklcgy`
- Litecoin([QR](/assets/qrcodes/litecoin.png)): `ltc1qc3unssu58qjrqdnnl8pxep9259khfwz46un2cd`
- BitcoinCash([QR](/assets/qrcodes/bitcoincash.png)): `qq9gvne3p7sa678j9y3y32ersju83elumclvknqm9h`
