---
title: "Antler - An IRL Browser"
description: "A specification for building mini apps that work with QR codes and open web standards."
date: "2025-11-25"
author: "Daniel Mathews"
author_image: "https://ax0.taddy.org/blog/about-us/danny-small-profile-pic.png"
author_url: "https://bsky.app/profile/dmathewwws.com"
app_id: "6753969350"
---

# Antler - An IRL Browser

Hey! I launched an app called [Antler](https://antlerbrowser.com). 

Antler started as a simple idea: A local internet just for my neighbourhood. However, the idea morphed into something a bit more practical: an open-source tool that helps developers build lightweight apps when **you know your users will be at a specific location.**

Here is a demo of Antler in action, I scan a QR code and immediately get checked into my co-working space.

<video src="https://ax0.taddy.org/dmathewwws/antler-demo.mp4" controls width="100%" autoplay muted loop></video>

However, that checkin app is just one example of a lightweight mini app, any developer can build their own mini app. For example, [draw-on-my-phone](https://drawing.dmathewwws.com) is a game where a group of friends scan a QR code to enter a game room and take turns drawing on each other's phones (similar to Telestrations or Garlic Phone)

In both cases, a user scans a QR code and instantly gets logged in. Moreover, no servers were needed to make this happen.

Antler is an iOS or Android app that users download. However, what makes Antler interesting is that it uses an open-specification called the [IRL Browser Specification](https://antlerbrowser.com/open-specification). This spec is an attempt to answer the question: **Can we build a mini app platform on open web standards?**

In a future where this specification is adopted, you can scan a QR at a coffee shop, concert, or conference → You instantly access the experience. No downloads. No signups. 

## WeChat Mini Apps

Currently, there are developer platforms that make it easier for developers to build lightweight mini apps:

- WeChat Mini Apps
- WhatsApp Business / WhatsApp Business API
- iOS App Clips

What makes it easier to build on these platforms is they take care of auth, payments and help make it easier to host / distribute your app. 

Let's focus on WeChat. You might know it as a popular messaging app in China similar to WhatsApp or Telegram. But it's also called a super app because of all the useful mini apps you can use inside it. 

For example, if you wanted to rent a bike in Shanghai, just open WeChat and scan the QR code on a bike to unlock and start riding. Under the hood, the bike share company’s mini app gets downloaded and runs inside WeChat, your WeChat id is used for registration/verification with the bike sharing app and WeChat pay is used for payment after the ride is complete.

![wechat-bike-share.png](https://ax0.taddy.org/dmathewwws/wechat-bike-share.png)

WeChat Mini apps proved out a really important use case in the Chinese market: You don’t always need a native app. Sometimes the better experience is to just scan a QR code.

## How Antler Works

Antler’s goal is to give the same great UX experience from WeChat mini apps, using regular QR codes + open web standards + an open specification used for communication. What makes Antler unique to WeChat is there is no central Antler server that is used for auth. This is how it works:

![antler-how-it-works.png](https://ax0.taddy.org/dmathewwws/antler-how-it-works.png)

When a user downloads Antler, they create a profile that is stored locally on their device.

A profile contains: 

- a [DID](https://www.w3.org/TR/did-1.0/) (a W3C standard for identity) - a public key
- a private key
- a name
- link to socials (optional)
- an avatar (optional)

When a user scans a QR code, Antler opens your website inside a WebView and injects a `window.irlBrowser` object.

The `window` object is available on all browsers, and as a developer it gives you access to useful browser features. For example, `window.location` lets you know the current url you are visiting in the browser. We made up a new property called `window.irlBrowser` and use it as an interface to communicate between the Antler app and your website.

Your website calls `window.irlBrowser.getProfileDetails()`  and gets back cryptographically signed profile data as a JWT.

```json
{
  "iss": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
  "aud": "https://yourdomain.com",
  "iat": 1728393600,
  "exp": 1728397200,
  "type": "irl:profile:details",
  "data": 
    {
      "did": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
      "name": "Danny Mathews",
      "socials": [{ "platform": "INSTAGRAM", "handle": "dmathewwws" }]
    }
}
```

You should decode and verify that the public key in the `iss` field was used to sign this data. This way you know only someone with the private key for this DID could have sent it.

And voila, the user is instantly logged into your website. Profile details that were stored locally on the user’s device are shared to your website and no servers were involved! 

![wechat-antler-comparison.png](https://ax0.taddy.org/dmathewwws/wechat-antler-comparison.png)

## Open Source

Antler is [open-source](https://github.com/antler-browser/antler). It's a simple React Native app that stores user profiles and public / private key pairs.

Antler uses an [open specification](https://antlerbrowser.com/irl-browser-standard.html) to pass data between your website and the mobile app. These are the five functions that are defined in the spec.

```tsx
interface IRLBrowser {
  // Get profile details (name, socials)
  getProfileDetails(): Promise<string>;
  
  // Get avatar as base64-encoded string
  getAvatar(): Promise<string | null>;
  
  // Get details about the IRL Browser
  getBrowserDetails(): BrowserDetails;
  
  // Request additional permissions (in the future)
  requestPermission(permission: string): Promise<boolean>;
  
  // Close the WebView (return to QR scanner)
  close(): void;
}
```

Being an open specification means anyone can create an alternative to Antler. 

It also means if you are a developer, and:

- Want to add instant login via a QR code to your website, you know you are not locked into Antler or a closed platform.
- Want to integrate mini apps into your current app. All the mini apps that work with Antler will work inside your app (just follow the spec).

### Do your users have to download an app?

Your users don’t actually need to download Antler or any mobile app. Here is a client side package [`irl-browser-onboarding`](https://github.com/antler-browser/irl-browser-onboarding) that you can add to your website that takes advantage of Antler being built on an open specification to create a Temporary / One Time account.

The package checks if your mini app is being viewed inside a IRL Browser, and if not, creates an onboarding flow where a user enters their name, social links, and avatar and injects the same `window.irlBrowser` API that Antler or any IRL Browser would. This means if users want they can use Antler and get the immediate login UX and a persistent profile, or they can create a one-time / temporary profile if don’t.

## Example Apps

My hope with IRL Browser mini apps is that it encourages developers like me to build apps that we would have never thought would be feasible to build a native app for ie) building an app for my social clubs, local community events, venues, pop-ups, game nights with friends, or any lightweight gathering where people are physically present.

Here are some example use cases:

- Networking
    - [Check into event app](https://github.com/antler-browser/meetup-cloudflare) (Demo: [https://zspace.dmathewwws.com](https://zspace.dmathewwws.com))
- IRL Games with friends
    - [Draw on my phone IRL game](https://github.com/antler-browser/draw-on-my-phone-game) (Demo: [https://drawing.dmathewwws.com](https://drawing.dmathewwws.com))
- Business Applications:
    - Loyalty Program for a coffee shop (you shouldn’t have to download a different app for every coffee shop)
- Interactive Arts Projects
- Community Building / Neighbourhood Projects
    - ex) Relational Tech Project: [https://relationaltechproject.org/remix](https://relationaltechproject.org/remix)
- Hopefully even more

## Branding

An interesting idea that came up while building this was to create a visual cue so a user knows if you scan this QR code it offers instant login. Antlers was what I came up with, and it inspired the app's name.

![antler-branding.png](https://ax0.taddy.org/dmathewwws/antler-branding.png)

## Future Roadmap For Antler

- Add NFC (doesn't have to be just QR codes)
- Add Incognito Mode
- Add more DID Methods, Add Verifiable Credentials.
- Pass through more native capabilities to mini apps (with explicit user permission)
    - Location
    - Push Notifications

## **Useful Resources**

[Scuttlebutt](https://scuttlebutt.nz/docs/protocol/) - If you care about the original idea of Antler, having a local internet just for your neighbourhood, the Scuttlebutt project is a really fun way I've seen people try to achieve this.

[Feathers Auth](https://feathers.dev/auth/docs): One of the inspirations behind Antler. The first time I saw a working demo of local-first auth was this [local-first chat app](https://github.com/DWebYVR/featherschat) built by [David](https://bsky.app/profile/daffl.xyz).

[IRL Browser Specification](https://antlerbrowser.com/open-specification) - The specification for how IRL Browsers communicate with mini-apps through DIDs and JWTs.

[DID](https://www.w3.org/TR/did-1.0/) - W3C standard for identities. Right now Antler just supports the key method, but there are other methods we could integrate with.

[Verifiable Credentials](https://www.w3.org/TR/vc-data-model-2.0/) - W3C standard that works with DIDs. It allows you to verify something is true without revealing unnecessary data ex) you could prove you own a ticket to a concert 

[WeChat MiniApps Docs](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/code.html) (in Chinese - but your browser can translate it for  you) 

[MiniApps Standard](https://www.w3.org/TR/mini-app-white-paper/) - A W3C Draft by competitors of WeChat (Alibaba, Baidu, Xiaomi) to create a standard for MiniApps that isn't tied to WeChat. A great way to deep dive into the architecture behind MiniApps.

[WeChat Strategy Doc](https://www.wechatwiki.com/wp-content/uploads/wechat-mini-program-light-app-report-fabernovel-31ten.pdf): A 326 page pdf on the different ways companies are using Mini Apps. It's a great resource.

[How Businesses in India Use WhatsApp](https://newsletter.theindianotes.com/p/whatsapp-owns-india): A in-depth blog on how businesses in India use WhatsApp.

[Farcaster Mini Apps](https://miniapps.farcaster.xyz) - Similar concept to WeChat MiniApps but integrated into the Farcaster social network. It implements a similar specification to IRL Browser ie) uses a WebView to communicate with mini apps and their app and mini apps are built with standard HTML, CSS, and Javascript.

[WebXDC](https://webxdc.org/docs/get_started.html) - A similar specification to IRL Browser as well. Focused on chat apps that want to integrate a mini app experience into their chat app. 

## Next Steps

Thanks for taking the time to read this deep dive! 

If you are a developer and:

- Want to explore what building a mini-app looks like, check out these [open-source mini apps](https://antlerbrowser.com/open-source).
- Have already built a chat / social app and want to integrate Mini Apps into your app, check out the [IRL Browser specification](https://antlerbrowser.com/irl-browser-specification).
- Want to chat? [Join our Discord](https://discord.gg/ksewAwnGsN)