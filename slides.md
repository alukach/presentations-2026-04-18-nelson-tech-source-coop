---
title: Source Cooperative — A Technical Walkthrough
info: |
  How we rebuilt Source Cooperative on Rust, WASM, and Cloudflare Workers
class: text-center
highlighter: shiki
drawings:
  persist: false
  enable: false
transition: slide-left
mdc: true
addons:
    - slidev-addon-qrcode

theme: './theme'
layout: title
image: /images/theme/lena-delta.jpg
---

# Source Cooperative

::subtitle::
Flat files, Cloudflare, and the S3 Protocol

<DecorativeRectangle
  width="50%"
  height="40%"
  zIndex=20
  :position="{
    bottom: '2%',
    right: '2%',
  }"
  :customStyle="{ mixBlendMode: 'multiply' }"
>
  <div w-full h-full relative flex flex-col items-end justify-end p-4 text-white text-right>
    <pre>
    Nelson Tech Meetup
    2026-04-22</pre>
    <h5 text-sm>
      <code text-primary>@alukach</code>
    </h5>
  </div>
</DecorativeRectangle>
<LogoHorPos position="top-left" height="24px" />

---
layout: iframe-right
url: https://developmentseed.org/
scale: 0.5
---

## About Me

<v-clicks depth="2">

* Anthony Lukach
* Cloud Engineer @ **Development Seed**
  * geospatial data consultancy
  * great partners: NASA, ESA, IFRC, MSFT PC
  * business model: build **open source** tools to enable **science**
* Current focus: **Source Cooperative** from Radiant Earth

</v-clicks>

<LogoHorNegMono position="bottom-right" />

---
layout: cover
background: /images/theme/landsat9-taklimakan-desert-china.jpg
dim: true
class: text-center
---

# Source Cooperative

## (A Radiant Earth project)

---
layout: iframe-right
url: https://radiant.earth/
scale: 0.5
---


## What Is Radiant Earth

`https://radiant.earth`

A **501(c)(3) non-profit**, based in Washington, DC.

<v-clicks>

* Two main initiatives:
  * **Source Cooperative** — open data publishing
  * **Cloud-Native Geospatial Forum** — community + education
* Funded by donations — **no investors, no shareholders, no one can own equity**

</v-clicks>

---
layout: iframe-right
url: https://source.coop/
scale: 0.5
---

## What is Source Cooperative

`https://source.coop`

Radiant Earth's flagship product, launched in 2023.

<v-clicks level=2>

* Built for **researchers, scientists, and engineers** publishing open data — without standing up their own portals, APIs, or servers
* **Cloud object storage + cloud-native formats** — datasets exceeding 100 TB, served as plain HTTP
* Mechanically:
  * a **unified namespace** across multiple object-storage backends
  * a **data proxy** for access control and routing
  * a **discovery UI** for published datasets

</v-clicks>

<div absolute bottom-0 left-0 flex items-center gap-2>
  <CurrentUrlQRCode
    width="120" height="120"
    image='/images/logos/symbol--pos-neg@2x.png'
    url="https://radiant.earth/blog/2023/10/what-is-source-cooperative/"
    :dotsOptions="{
      type: 'dots',
      color: 'var(--slidev-theme-primary)'
    }"
  />
  <span text-xs italic>
  ☜ "What Is Source Cooperative"
  </span>
</div>


---
layout: image-right
# Landsat 9 image of Bangladesh Coast
image: /images/theme/landsat9-bangladesh-coast.jpg
class: image-narrow
---

## Value proposition

> So, you're just skinning S3?


---
layout: image-right
# Landsat 9 image of Bangladesh Coast
image: /images/theme/landsat9-bangladesh-coast.jpg
class: image-narrow
---

## Scale of usage

<v-clicks>

* **>4 PB** of data hosted
* **92 TB/day** distributed in the month to date (**~8.47 Gbps** throughput)
* **1.7M req/day** over past 30 days
  * Bursts from **~25 req/sec** to **~500 req/sec**

<SlidevVideo autoplay controls>
  <!-- Anything that can go in an HTML video element. -->
  <source src="/images/bursty.mp4" type="video/mp4" />
</SlidevVideo>

</v-clicks>

<br>

<div v-click text-sm italic opacity-60>
Numbers to ground the rest of the talk.
</div>

---
layout: cover
background: /images/theme/landsat8-ord-river-australia.jpg
dim: true
class: text-center
---

# Part 2

The Substrate

---
layout: image-right
# Landsat 8 image of the Ord River in Australia
image: /images/theme/landsat8-ord-river-australia.jpg
class: image-narrow
---

## Object Storage + Cloud-Native Formats

<v-clicks>

* **S3** made range-addressable blob storage cheap and ubiquitous
* **Cloud-native formats** — COG, PMTiles, GeoParquet, Zarr — exploit range requests to push computation client-side
* No database tier. No API server. **Just bytes laid out intelligently.**

</v-clicks>

---
layout: image-right
# Landsat 9 image of Taklimakan Desert, China
image: /images/theme/landsat9-taklimakan-desert-china.jpg
class: image-narrow
---

## What This Looks Like in Practice

<v-clicks>

* Short demo: PMTiles range requests in browser DevTools
* You can literally **watch the architecture work**
* Source Coop's job: be the access layer for this pattern at scale

</v-clicks>

<div v-click absolute bottom-8 left-8 right-8 p-4 border border-dashed border-primary text-xs opacity-70>
  📺 placeholder: recorded GIF of DevTools network panel
</div>

---
layout: cover
background: /images/theme/landsat9-kangerdlugssuaq-greenland.jpg
dim: true
class: text-center
---

# Part 3

From v1 to v2

---
layout: image-right
# Landsat 8 image of Ayon Island
image: /images/theme/landsat8-ayon-island.jpg
class: image-narrow
---

## v1 Architecture

<v-clicks>

* Single-region deployment
* ALB-fronted proxy
* Backed by S3

</v-clicks>

<div v-click absolute bottom-8 left-8 right-8 p-4 border border-dashed border-primary text-xs opacity-70>
  📐 placeholder: v1 request-path diagram
</div>

---
layout: image-right
# Greenland fjords abstract
image: /images/theme/blue-white-red-abstract-painting.jpg
class: image-narrow
---

## Why v1 Had to Change

<v-clicks>

* **Egress costs** through ALB dominated the bill
* **Single distribution point** → latency for global users
* **Scaling ceiling** on the proxy tier

</v-clicks>

---
layout: cover
background: /images/theme/landsat8-klyuchevskaya-kamchatka.jpg
dim: true
class: text-center
---

# The Bet

Rust → WASM → Cloudflare Workers

---
layout: image-right
# Landsat 8 image of Klyuchevskaya, Kamchatka
image: /images/theme/landsat8-klyuchevskaya-kamchatka.jpg
class: image-narrow
---

## The Bet

<v-clicks>

* Rewrite the proxy to compile to **WASM**
* Deploy on **Cloudflare Workers**
* Three slides on *why* follow

</v-clicks>

---
layout: image-right
# Landsat 9 image of Western Guinea-Bissau
image: /images/theme/landsat9-western-guinea-bissau.jpg
class: image-narrow
---

## Why Cloudflare Workers

<v-clicks>

* **No egress fees** inside the Cloudflare network
* Globally distributed with optimized routing
* Near-zero cold start
* Serverless operational model

</v-clicks>

---
layout: image-right
# Sentinel-2A image of the Southern Tibetan Plateau
image: /images/theme/sentinel2a-southern-tibetan-plateau.jpg
class: image-narrow
---

## Why Rust → WASM

<v-clicks>

* Performance and predictable memory behavior
* Type safety for a proxy handling untrusted input
* First-class WASM target via `workers-rs`

</v-clicks>

---
layout: image-right
# Copernicus Sentinel-2 image of Tanezrouft Basin
image: /images/theme/Tanezrouft_Basin.jpg
class: image-narrow
---

## The Constraint That Shaped Everything

<v-clicks>

* Workers' **CPU time limits** per request
* Can't **buffer** the response body
* Can't **re-sign or mutate** payloads in flight
* Must **stream bytes through, zero-copy, end to end**

</v-clicks>

---
layout: image-right
# Landsat 9 image of Apostle Islands, Lake Superior
image: /images/theme/landsat9-apostle-islands-lake-superior.jpg
class: image-narrow
---

## Why We Built `multistore`

<v-clicks>

* Existing S3 SDKs assume buffering and mutation are free
* We needed an S3-compatible API library that:
  * Streams without buffering
  * Dispatches across multiple storage backends
  * Stays inside Workers' CPU budget

</v-clicks>

---
layout: image-right
# Landsat 9 image of Kangerdlugssuaq Glacier, Greenland
image: /images/theme/landsat9-kangerdlugssuaq-greenland.jpg
class: image-narrow
---

## Architecture Diagram (v2)

<v-clicks>

* `Client` → Cloudflare edge → Worker → `multistore` → backend (S3, R2, …)
* Annotate where **SigV4**, **routing**, and **streaming** happen

</v-clicks>

<div v-click absolute bottom-8 left-8 right-8 p-4 border border-dashed border-primary text-xs opacity-70>
  📐 placeholder: v2 architecture diagram
</div>

---
layout: cover
background: /images/theme/satellite-image-body-of-water.jpg
dim: true
class: text-center
---

# Part 4

What Else Workers Gave Us

---
layout: image-right
# Landsat 9 satellite image of Prudhoe Bay, Alaska
image: /images/theme/satellite-image-body-of-water.jpg
class: image-narrow
---

## What Else Workers Gave Us

A brief feature tour — 30–60 sec each on the next slides.

<v-clicks>

* Durable Objects
* Analytics Engine
* R2 + R2 Pipelines

</v-clicks>

---
layout: image-right
# Landsat 9 image of Bangladesh Coast
image: /images/theme/landsat9-bangladesh-coast.jpg
class: image-narrow
---

## Durable Objects

<v-clicks>

* Used for **websocket fan-out**
* Powers our real-time **request globe** visualization

</v-clicks>

---
layout: image-right
# Landsat 8 image of the Ord River in Australia
image: /images/theme/landsat8-ord-river-australia.jpg
class: image-narrow
---

## Analytics Engine

<v-clicks>

* Per-request telemetry **without standing up a pipeline**
* Quirks: limited SQL dialect

</v-clicks>

---
layout: image-right
# Landsat 9 image of Taklimakan Desert, China
image: /images/theme/landsat9-taklimakan-desert-china.jpg
class: image-narrow
---

## R2 + R2 Pipelines

<v-clicks>

* Storage backend with **no egress fees**
* Pipelines for **analytics ingestion into Parquet**

</v-clicks>

---
layout: cover
background: /images/theme/landsat8-ayon-island.jpg
dim: true
class: text-center
---

# Part 5

Operations & Open Questions

---
layout: image-right
# Landsat 8 image of Ayon Island
image: /images/theme/landsat8-ayon-island.jpg
class: image-narrow
---

## Operational Costs

<v-clicks>

* **Old stack vs. new stack** — concrete $/month or $/TB
* What dominates the bill: **observability events**, not compute or egress
* Key tuning knob: `head_sampling_rate`

</v-clicks>

---
layout: image-right
# Greenland fjords abstract
image: /images/theme/blue-white-red-abstract-painting.jpg
class: image-narrow
---

## Failure Modes & Ongoing Constraints

<v-clicks>

* Where the system **bends**
* What we **monitor** and why

</v-clicks>

---
layout: image-right
# Landsat 8 image of Klyuchevskaya, Kamchatka
image: /images/theme/landsat8-klyuchevskaya-kamchatka.jpg
class: image-narrow
---

## Open Questions

<v-clicks>

* **OIDC workload identity** for backend access
* **STS session tokens** for scoped credentials
* **Metered access** patterns for billing and quotas

</v-clicks>

<br>

<div v-click text-sm italic opacity-60>
Still figuring this out — discussion welcome.
</div>

---
layout: image-right
# Sentinel-2A image of the Southern Tibetan Plateau
image: /images/theme/sentinel2a-southern-tibetan-plateau.jpg
class: image-narrow
---

## Links & References

* **Source Cooperative** — [source.coop](https://source.coop)
* **multistore** — github.com/source-cooperative/multistore
* **"What Is Source Cooperative"** — source.coop/blog
* **Me** — `@alukach` on GitHub / Bluesky

<div absolute bottom-8 left-8 flex items-end gap-2>
  <CurrentUrlQRCode
    width="120" height="120"
    image='/images/logos/symbol--pos-neg@2x.png'
    url="https://source.coop"
    :dotsOptions="{
      type: 'dots',
      color: 'var(--slidev-theme-primary)'
    }"
  />
  <span text-xs mb-2 italic>
  ☜ source.coop
  </span>
</div>

---
layout: title
# Landsat 9 image of Apostle Islands, Lake Superior
image: /images/theme/landsat9-apostle-islands-lake-superior.jpg
---

# Questions?

<DecorativeRectangle
  width="30%"
  height="96%"
  zIndex=11
  :position="{
    bottom: '2%',
    right: '2%',
  }"
  :customStyle="{ mixBlendMode: 'multiply' }"
>
  <div w-full h-full relative flex flex-col items-start justify-between p-4 text-white text-left class="[&_a]:no-underline [&_a]:text-white [&_a:hover]:text-gray-200">
    <div mb-4 flex flex-col gap-5 items-start justify-start text-sm font-mono class="[&_a]:flex [&_a]:items-center [&_a]:gap-1">
      <Logo src="/images/logos/hor--neg-mono@2x.png" height="24px" alt="DevelopmentSeed" class="!relative !top-0 !left-0" />
      <a href="https://developmentseed.org" target="_blank" title="Website">
        <WebsiteIcon size="20" pr-1 />
        <span>developmentseed.org</span>
      </a>
      <a href="mailto:hello@developmentseed.org" title="Email">
        <EmailIcon size="20" pr-1 />
        <span>hello@developmentseed.org</span>
      </a>
      <a href="https://github.com/developmentseed" target="_blank" title="GitHub">
        <GitHubIcon size="20" pr-1 />
        <span>@developmentseed</span>
      </a>
      <a href="https://www.linkedin.com/company/development-seed" target="_blank" title="LinkedIn">
        <LinkedInIcon size="20" pr-1 />
        <span>development-seed</span>
      </a>
      <CurrentUrlQRCode
        fullWidth
        image='/images/logos/symbol--neg-mono@2x.png'
        :dotsOptions="{ type: 'classy-rounded', color: 'white' }"
      />
    </div>
    <div opacity-70 w-100 class="text-[10px]">
      <div>Attributions:</div>
      <div>
        Slide images from <a href="https://unsplash.com/@usgs?utm_source=ds-slides&utm_medium=referral" target="_blank" class="text-white hover:text-gray-200">USGS</a> on <a href="https://unsplash.com/?utm_source=ds-slides&utm_medium=referral">Unsplash</a>
      </div>
    </div>
  </div>
</DecorativeRectangle>
