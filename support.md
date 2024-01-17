---
layout: homepage
title: Support
---

## Support

If you appreciate the free and open-source projects and content I provide and would like to support my work, you can make a donation. Your generosity helps me continue creating and maintaining these projects and resources.

### Cryptocurrency Donations

{% for crypto in site.data.crypto_donations %}

- **{{ crypto.name }}** _([QR]({{ crypto.qr }}))_
  ```
  {{ crypto.address }}
  ```

{% endfor %}

Your support means a lot and enables me to keep contributing to the free and open-source community and create content. Thank you for considering it!
