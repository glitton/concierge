# Front End Performance Auditing

## This page shares best practices for Gatsby Performance Audits

Customer information has been removed to protect their privacy

---

# **Audit Lessons**

## Understand baseline data using [webpagetest.org](http://webpagetest.org) and treo.sh

The content here is based on [this interview with Harry Roberts](https://www.twitch.tv/videos/1435367315?t=00h21m09s) (start at around 21:09)

When starting audit the first thing to do is run the url such as [construction.XXX](http://construction.XXX) in [webpagetest.org](http://webpagetest.org) and treo.sh

- [webpagetest.org](http://webpagetest.org) desktop settings - click on Advanced Configurations, choose the following:
  - Test location: Virginia
  - Browser: Chrome
  - Choose the Test Setting tab
    - Connection: Cable (5/1 Mbps 28ms RTT)
    - Desktop Browser Dimensions: default (1366x768)
    - Number of Tests to Run: 7
    - Repeat View: First View Only
    - Check Capture Video
    - Optional: Add text to the label that describes the test i.e. home page with no Google Tag Manager
- [webpagetest.org](http://webpagetest.org) mobile settings
  - Test location: Virginia
  - Browser: Motorola G (gen 4)
  - Choose the Test Setting tab
    - Connection: 3G Fast
    - Desktop Browser Dimensions: default (1366x768)
    - Number of Tests to Run: 7
    - Repeat View: First View Only
    - Check Capture Video
    - Optional: Add text to the label that describes the test i.e. home page with no scripts
  - Using [treo.sh](http://treo.sh). This tool is Chrome specific and shows [CRUX data](https://web.dev/chrome-ux-report/#:~:text=The%20Chrome%20UX%20Report%20(informally,in%20users%20in%20the%20field.)
    - Go to [https://treo.sh/sitespeed](https://treo.sh/sitespeed)
    - Enter the domain such as [construction.XXX](http://construction.XXX) ([https://treo.sh/sitespeed/construction.autodesk.com](https://treo.sh/sitespeed/construction.autodesk.com))
    - Example screenshot and observations below

![Look at the deltas between Time to First Byte and First Contentful paint.  The delta of 700ms means there is that amount of blocking time before the first paint is rendered.  What’s more telling is the delta between First Contentful Paint and Largest Contentful paint.  There is 2.7s of blocking time likely attributed to JavaScript or images, etc ... [View video here](https://www.twitch.tv/videos/1435367315?t=00h25m01s) for a full analysis](XXX)

Look at the deltas between Time to First Byte and First Contentful paint. The delta of 700ms means there is that amount of blocking time before the first paint is rendered. What’s more telling is the delta between First Contentful Paint and Largest Contentful paint. There is 2.7s of blocking time likely attributed to JavaScript or images, etc ... [View video here](https://www.twitch.tv/videos/1435367315?t=00h25m01s) for a full analysis

> “JavaScript is the fastest way to build a slow website” - Harry Roberts

- Example output and analysis of webpagetest from the video can be seen starting from [here](https://www.twitch.tv/videos/1435367315?t=00h29m24s).
- Compare each of the webpage results to see if there are patterns, such as if the all pages have the same head tags which makes the start render time the same.
- Large LCP symptoms

### Tips

- Elements in Chrome is the DOM
- Sources tab is the actual HTML
- If you are using javascript to build your page, do not defer it. Good example on how to [view the source is at 1:01:11](https://www.twitch.tv/videos/1435367315?t=01h01m11s)
- If you want to see how much javascript is needed to build the site, disable javascript as described [here](https://developer.chrome.com/docs/devtools/javascript/disable/) and re-render the page
- The horizontal lines in the Chrome dev tools network tab represent various domains. Details [here](https://www.twitch.tv/videos/1435367315?t=01h10m47s).
-

## Overall Performance

- Avoid passing large queries into context in the createPages API of XXX or you will get context very large that your build and performance will suffer. Example is XXX

![XXX.png](XXX)

# FCP

1. Symptoms: No content in initial HTML due to the site not rendering HTML, HTML rendering relies on client-side JavaScript
   1. Fix - check XXX.js and XXX.js to see if the wrapRootElement is affecting HTML being built during build time with Redux. Don’t use Redux!

- Inline SVGs instead of using img tags with SVgs especially for images above the fold
- Add woff fonts to header, key feature in XXX
- ***

# LCP

- Load hero images in the same domain and not in the CMS so to not make a network request
- If your LCP is a background image, see this [hack](https://www.twitch.tv/videos/1435367315?t=01h19m19s).
- Text is the fastest LCP
- Avoid fade ins for LCP
- Do not lazy load LCP
- Sometimes a Cookie modal loads and is deemed as the LCP, be aware of this

---

# TBT

- If on XXX, use the [XXX component](https://www.XXX) to reduce impact of third party scripts
- Load GTM using onInitialClientRender in [XXX file](https://github.com/XXX). Doing this definitely reduced TBT for XXX but other customers will need to try this out and make sure that GTM is still calling the other scripts. If you use this, comment out gatsby-plugin-google-tagmanager in gatsby config. **_<Note the code below is a WIP, it isn’t working as intended just yet>_**

  ```jsx
  // gatsby-browser only supports common js modules

  const GID = "<put in GTM-ID from customer>";
  const dataLayer = (window.dataLayer = window.dataLayer || []);
  function gtag() {
    dataLayer.push(arguments);
  }

  exports.onInitialClientRender = () => {
    const gtagScript = document.createElement("script");
    gtagScript.src = "https://www.googletagmanager.com/gtag/js?id=${GID}";
    document.head.appendChild(gtagScript);

    exports.onRouteUpdate = ({ location }) => {
      gtag("set", "page_path", location.pathname);
      gtag("event", "page_view");
    };
  };
  ```

- Use native fetch instead of the axios library

### Using the Script API off-the-main-thread strategy

Since the release of the XXX API, we’ve had a lot of success using the [XXX](XXX) strategy for third party scripts such as Google Tag Manager. Since Google Tag Manager is responsible for firing other scripts, more setup is required than just adding the script tag and a default strategy. You will need to add the forward collection, proxy configuration and resolving URLs as explained in the documentation.

Here’s an [example implementation](https://github.com/XXX) from a customer. Specifically take a look at the following:

- [XXX](XXX) - line 12 shows the code for forward collection. Lines 14 to 19 is the google tag manager initialization event which I am not sure is needed. It doesn’t hurt for it to be there.
- [XXX](XXX) - use wrapPageElement to implement gatsby-shared.js
- gatsby-ssr.js - [XXX](XXX) to implement gatsby-shared.js.
- proxy configuration - [XXX](XXX), the XXX is an array that contains the URLs of the scripts that google tag manager fires.
- [XXX](XXX) in gatsby-ssr.js - each proxied URL needs to have a resolve URL block as seen in lines [XXX](XXX)

### SVGs

**Symptoms**

- lots of SVGs in app.js bundle

Don’t import SVGs as Javascript, because they:

- end up the bundle which makes it larger and increase network/parsing time
- increase the DOM size which hurts hydration and TBT
- [https://twitter.com/MarkSShenouda/status/1491510779685789697](https://twitter.com/MarkSShenouda/status/1491510779685789697)
- [https://benadam.me/thoughts/react-svg-sprites/](https://benadam.me/thoughts/react-svg-sprites/)

---

- De-prioritize JavaScript. Using [this code in XXX](https://github.com/XXX), moved JS execution later in the waterfall but it didn’t change any of the core web vitals for the customer. It may work for others so good to give this a try.

```jsx
export const onPreRenderHTML = ({
  getHeadComponents,
  replaceHeadComponents,
}) => {
  const head = getHeadComponents().filter((cmp) => {
    return cmp.props.as !== "script";
  });
  replaceHeadComponents(head);
};
```

# CLS

- In mobile, animation or CSS causing layout shifts, go back to basics and use media queries for Mobile. Load fonts locally so there isn’t a big shift
