---
title: "Can we build WeChat Mini Apps using open web standards?"
url-slug: "local-first-mini-apps-specification"
description: "A specification for building mini apps that work with QR codes and open web standards."
date: "2026-01-15"
author: "Daniel Mathews"
author_image: "https://ax0.taddy.org/blog/about-us/danny-small-profile-pic.png"
author_url: "https://bsky.app/profile/dmathewwws.com"
app_id: "6753969350"
---

You might have heard of WeChat, it's a popular messaging app in China similar to WhatsApp or Telegram. In this deep dive, I'm going to cover why WeChat Mini Apps are such a big deal in China, and showcase how we can build a similar experience using open web standards without relying on a super app like WeChat.

WeChat isn't just a popular messaging app in China, it's practically unavoidable. Even [tourists are advised to download it](https://www.scmp.com/lifestyle/travel-leisure/article/3230794/best-apps-foreign-visitors-china-and-which-are-useless-google-maps) before visiting. WeChat allows users to access a wide variety of services like food delivery, ride hailing, bike rentals, and more without leaving the app. This is why WeChat is called a "super app".

For example, if you wanted to rent a bike in Shanghai, just open WeChat and scan the QR code on the bike to unlock and start riding (no downloads, no signups). Under the hood, the bike service's mini app is downloaded but runs inside WeChat. Your WeChat ID is used for registration/verification and WeChat Pay is used for payment after the ride is complete.

![wechat-bike-share.png](https://ax0.taddy.org/dmathewwws/wechat-bike-share.png)

WeChat Mini apps proved out a really important use case in the Chinese market: You don’t always need a native app. Sometimes the better experience is to just scan a QR code.

However, outside China, while we have popular apps, we don't have a super app like WeChat. On one hand, we are missing out on the convenience of having a mini app ecosystem with instant login / payments. On the other hand, I don't think we would or should pick one company to become our WeChat. 

So, the question is: **Can we have a similar user experience to WeChat mini apps without relying on a super app like WeChat?** Yes! We can do this by using regular QR codes and open web standards.

## Quick Demo

I built a demo app called [Antler](https://antlerbrowser.com). When I open up Antler and scan a QR code, I immediately get checked into my co-working space.

<video src="https://ax0.taddy.org/dmathewwws/antler-demo.mp4" controls width="100%" autoplay muted loop></video>

What's interesting is that:
  1) No servers are needed to make this happen
  2) Antler uses an open-specification called the [Local First Auth Specification](https://antlerbrowser.com/local-first-auth-specification), which means anyone can build this into their app. 

## How Antler Works

What makes Antler unique to WeChat is there is no central Antler server that is used for auth. This is how it works:

![antler-how-it-works.png](https://ax0.taddy.org/dmathewwws/antler-how-it-works.png)

When a user downloads Antler, they create a profile that is stored locally on their device.

A profile contains:

- a [DID](https://www.w3.org/TR/did-1.0/) (a W3C standard for identity) - a public key
- a private key
- a name
- link to socials (optional)
- an avatar (optional)

When a user scans a QR code, Antler opens your website (mini app) inside a WebView and injects a `window.localFirstAuth` object.

The `window` object is available on all browsers and gives developers access to useful browser features. For example, `window.location` lets you know the current url you are visiting in the browser. We created a new property called `window.localFirstAuth` and use it as an interface to communicate between the Antler app and your mini app.

Your mini app calls `window.localFirstAuth.getProfileDetails()` and gets back cryptographically signed profile data as a JWT.

```json
{
  "iss": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
  "aud": "https://yourdomain.com",
  "iat": 1728393600,
  "exp": 1728397200,
  "data": {
    "did": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
    "name": "Danny Mathews",
    "socials": [{ "platform": "INSTAGRAM", "handle": "dmathewwws" }]
  }
}
```

You should decode and verify that the public key in the `iss` field was used to sign this data. This way you know only someone with the private key for this DID could have sent it.

And voila, the user is instantly logged into your mini app. Profile details that were stored locally on the user’s device are shared to your mini app and no servers were involved!

## Long Term Vision

Using an open specification means anyone can integrate this mini apps ecosystem into their app (i.e., any app that already has a user's profile) can be a Local First Auth app for mini apps. Moreover, any developer can build a mini app that is compatible with any Local First Auth app.

![irl-browser-vision.png](https://ax0.taddy.org/dmathewwws/irl-browser-vision-4.png)

As a developer, building a mini app should be a lot easier to build than a native app. I hope this encourages developers to build apps that we would not be practical to build a native app for ie) building an app for my social clubs, local community events, venues, pop-ups, game nights with friends, or any lightweight gathering where people are physically present.

Here are some example mini apps that you can build:

- Networking
  - eg) [Check into event app](https://zspace.dmathewwws.com). Code: https://github.com/antler-browser/meetup-cloudflare
- IRL Games with friends
  - eg) [Draw on my phone IRL game](https://drawing.dmathewwws.com). Code: https://github.com/antler-browser/draw-on-my-phone-game
  - Scavenger Hunts (go around looking for QR codes)
- Business Applications:
  - Loyalty Program for a coffee shop (you shouldn’t have to download a different app for every coffee shop)
- Community Building / Neighbourhood Projects
  - eg) Relational Tech Project: [https://relationaltechproject.org/remix](https://relationaltechproject.org/remix)
- Interactive Arts Projects

Lastly, in a future where this specification is adopted by multiple Local First Auth apps, a user can choose which app they want to use to scan a QR code and scan a QR code at a coffee shop, concert, or conference → You instantly access the experience. No downloads. No signups.

## Do users have to download an app to use mini apps?

When demoing Antler to some friends, I noticed some of them were hesitant to download another app on their phone. We can take advantage of the Local First Auth Specification being an open specification to create a temporary / one time account that doesn't require an app to be downloaded.

Here is a client side package [`local-first-auth`](https://github.com/antler-browser/local-first-auth) that you can add to your mini app. The package checks if your mini app is being viewed inside a Local First Auth app, and if not, creates an onboarding flow and injects those details into the `window.localFirstAuth` API that Antler or any Local First Auth app would. 

This means if users doesn't want to download an app, they can create a one-time / temporary profile just for your mini app, but if they download a Local First Auth app, they get an immediate login UX and a persistent profile they can use across all mini apps.

## Open Source

Antler is [open-source](https://github.com/antler-browser/antler). It's a simple React Native app that stores user profiles and public / private key pairs.

Antler uses an [open specification](https://antlerbrowser.com/local-first-auth-specification) to pass data between your mini app and the mobile app. These are the five functions that are defined in the spec.

```tsx
interface LocalFirstAuth {
  // Get profile details (name, socials)
  getProfileDetails(): Promise<string>;

  // Get avatar as base64-encoded string
  getAvatar(): Promise<string | null>;

  // Get details about the Local First Auth app
  getAppDetails(): AppDetails;

  // Request additional permissions (in the future)
  requestPermission(permission: string): Promise<boolean>;

  // Close the WebView (return to QR scanner)
  close(): void;
}
```

Being an open specification means anyone can integrate this mini apps ecosystem into their app. i.e) any app can be a Local First Auth app and all the mini apps that work with Antler will work inside your app (just follow the spec).

## **Useful Resources**

[Feathers Auth](https://feathers.dev/auth/docs): One of the inspirations behind Antler. The first time I saw a working demo of local-first auth was this [local-first chat app](https://github.com/DWebYVR/featherschat) built by [David](https://bsky.app/profile/daffl.xyz).

[Local First Auth Specification](https://antlerbrowser.com/local-first-auth-specification) - The specification for how Local First Auth apps communicate with mini-apps through DIDs and JWTs.

[DID](https://www.w3.org/TR/did-1.0/) - W3C standard for identities. Right now Antler just supports the key method, but there are other methods we could integrate with.

[Verifiable Credentials](https://www.w3.org/TR/vc-data-model-2.0/) - W3C standard that works with DIDs. It allows you to verify something is true without revealing unnecessary data ex) you could prove you own a ticket to a concert

[WeChat MiniApps Docs](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/code.html) (in Chinese - but your browser can translate it for  you)

[MiniApps Standard](https://www.w3.org/TR/mini-app-white-paper/) - A W3C Draft by competitors of WeChat (Alibaba, Baidu, Xiaomi) to create a standard for MiniApps that isn't tied to WeChat. A great way to deep dive into the architecture behind MiniApps.

[WeChat Strategy Doc](https://www.wechatwiki.com/wp-content/uploads/wechat-mini-program-light-app-report-fabernovel-31ten.pdf): A 326 page pdf on the different ways companies are using WeChat. It's a great resource.

[How Businesses in India Use WhatsApp](https://newsletter.theindianotes.com/p/whatsapp-owns-india): A in-depth blog on how businesses in India use WhatsApp. The closest thing to a super app outside of China.

[Farcaster Mini Apps](https://miniapps.farcaster.xyz) - Similar concept to WeChat MiniApps but integrated into the Farcaster social network. It implements a similar specification to Local First Auth ie) uses a WebView to communicate with mini apps and their app and mini apps are built with standard HTML, CSS, and Javascript.

[WebXDC](https://webxdc.org/docs/get_started.html) - A similar specification to Local First Auth as well. Focused on chat apps that want to integrate a mini app experience into their chat app.

## Next Steps

Thanks for taking the time to read this deep dive!

If you are a developer and:

- Want to explore what building a mini-app looks like, check out these [open-source mini apps](https://antlerbrowser.com/open-source).
- Have already built a chat / social app and want to integrate Mini Apps into your app, check out the [Local First Auth specification](https://antlerbrowser.com/local-first-auth-specification).
- Want to chat? [Join our Discord](https://discord.gg/ksewAwnGsN)

And I want to leave you with this graphic that compares WeChat Mini Apps vs using open web standards to build mini apps.

![wechat-antler-comparison.png](https://ax0.taddy.org/dmathewwws/wechat-antler-comparison.png)