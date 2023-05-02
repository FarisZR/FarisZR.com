---
title: Did the EU ban JSdelivr and other free CDNs?
categories:
  - Privacy
tags:
  - JSdelivr GDPR
  - Google Fonts GDPR
  - free CDNs and GDPR
  - is JSdelivr GDPR compliant?
date: 2023-05-02
image: thumbnail.jpeg
slug: eu-gdpr-jsdelivr-free-cdn-ban
summary: You maight want to think twice about using free CDNs, the google fonts ban in Germany might affects the legality of CDNs like JSdelivr and Bunny fonts.
keywords:
  - Jsdelivr
  - Google Fonts
  - Bunny fonts
  - GDPR
---

So while I was updating my website, I was thinking about using JSdelivr for my blogs images, the slowest part of my website since it's hosted on a VPS and not on Cloudflare Pages to avoid further internet centralization.

But I remembered that a court in Germany recently ruled that using Google Fonts is illegal, does this affect JSdelivr and other public CDNs? Apparently it does.

**I'm not a lawyer, these are just my observations.**

## Data processing agreements with free CDNs

According to [legalweb.io](https://legalweb.io/en/news-en/cdns-with-gdpr-in-mind/), in order to use a CDN you need to have an data processing agreement, which you don't have with free CDNs, and the data needs to be processed within the EU or a country that is deemed to follow GDPR standards, which is certainly not the U.S. 

So basically any CDN provider that you don't have an Article 28 GDPR agreement with is illegal, any of the free CDNs like Jsdelivr are banned, that should include [Bunny fonts](https://fonts.bunny.net/)!, which was created as a GDPR-friendly alternative to Google Fonts, but since you don't have a DPA with them, it's still probably illegal.

## Legitimate interest and CDNs

In that [LG München ruling on Google fonts](https://rewis.io/urteile/urteil/lhm-20-01-2022-3-o-1749320/), it was also mentioned that the website had no legitimate interest in using X fonts (presumably Google fonts), since the website admin could easily host the fonts themselves.

> 3. Es liegt auch kein Rechtfertigungsgrund für den Eingriff in das allgemeine Persönlichkeitsrecht vor. Ein berechtigtes Interesse der Beklagten i.S.d. Art. 6 Abs. 1 f) DS-GVO, wie von ihr behauptet, liegt nicht vor, denn X. Fonts kann durch die Beklagte auch genutzt werden, ohne dass beim Aufruf der Webseite eine Verbindung zu einem X.-Server hergestellt wird und eine Übertragung der IP-Adresse der Webseitennutzer an X. stattfindet.

What about other use cases like video hosting, which is much more bandwidth intensive and benefits a lot from CDNs? the answer won't really matter as all your favourite streaming providers will have their own DPAs with their CDN providers, nobody's giving you a free video CDN!

Looks like I'll just stick to hosting my blog images on my free VPS (shout out to Oracle Free Tier), and if I really want to improve the speed of my blog, I should just move to a CDN host like Cloudflare pages.