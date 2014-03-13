---
layout: index
---

# Privacy starts in your home!

[kinko.me](https://kinko.me) implements an en/decrypting SMTP- and IMAP-proxy on ARM-class hardware, the kinko box. Emails are synced from the users' email accounts via IMAP to the box and are stored in plaintext in a secure storage area on the box. The kinko box also includes a webmailer to be able to use email with the browser.

Connections to the kinko box are secured by TLS using a private key only known to the box itself. Furthermore, the kinko box is tunnelled to a public internet location. Consequently, users can access secure email from everywhere, using IMAP compatible email clients and/or browsers, including mobile clients.

kinko uses GnuPG for encryption, with the addition of encrypting the email subject. Further additions should allow "Post-email alternatives" (a la bitmessage) to be used with the email clients that users are using today already. Other, privacy-related additions are planned as well.

Both the kinko box software and the routing service's software are open sourced.

