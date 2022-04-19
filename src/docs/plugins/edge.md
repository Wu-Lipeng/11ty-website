---
eleventyNavigation:
  key: Edge
  order: -2
  excerpt: A plugin to run Eleventy in an Edge Function to add dynamic content to your Eleventy sites.
# communityLinksKey: edge
overrideCommunityLinks: true
---
# Eleventy Edge {% addedin "2.0.0" %}

{{ eleventyNavigation.excerpt }}

{% callout "info" %}This feature is considered <strong>experimental</strong> and requires Eleventy <code>v2.0.0-canary.6</code> or higher. Our first release is limited to <a href="https://docs.netlify.com/netlify-labs/experimental-features/edge-functions/">Netlify Edge Functions</a> support only.{% endcallout %}

Eleventy Edge is an exciting new way to add dynamic content to your Eleventy templates. With a simple Eleventy shortcode you can opt-in a part of your Eleventy template to run on an Edge server, allowing your site to use dynamic, user-specific content!

Here are a few ideas:

* Any user personalized content (User accounts, premium-only content, AB testing)
* Accessing/setting HTTP Headers (e.g. Cookies, Save-Data, Client Hints, etc)
* [Handling Forms](https://demo-eleventy-edge.netlify.app/forms/)
* Using Geolocation information to localize content
* A zero-clientside JavaScript [Dark mode/Light mode toggle](https://demo-eleventy-edge.netlify.app/appearance/)

## Try out the demos

Try out the [Eleventy Edge demos using Netlify’s Edge Functions](https://demo-eleventy-edge.netlify.app/).

Read the [`demo-eleventy-edge` Source Code on GitHub](https://github.com/11ty/demo-eleventy-edge)

## How does it work?

Don’t already have an Eleventy project? Let’s go through the [Getting Started Guide first](/docs/getting-started/) and come back here when you’re done!

### 1. Installation

The Eleventy Edge plugin is bundled with Eleventy, but do note that the plugin requires version `2.0.0-canary.6` or newer.

At time of initial launch, you will need to use Netlify CLI to run Eleventy Edge locally (`netlify-cli` version `10.0.0` or higher).

```
npm install netlify-cli
```

### 2. Add to your configuration file

```js
const { EleventyEdgePlugin } = require("@11ty/eleventy");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(EleventyEdgePlugin);
};
```

<details>
<summary>Expand to read about the advanced options (you probably don’t need these)</summary>

```js
const { EleventyEdgePlugin } = require("@11ty/eleventy");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(EleventyEdgePlugin, {
    // controls the shortcode name
    name: "edge",

    // controls where the Edge Function bundles go
    functionsDir: "./netlify/edge-functions/",

    // Version check for the Edge runtime
    compatibility: ">=2",
  });
};
```

</details>

### 3. Create your Edge Function

Save this file to `./netlify/edge-functions/eleventy-edge.js`. Note that [Edge Functions](https://docs.netlify.com/netlify-labs/experimental-features/edge-functions/) run in Deno so they require ESM (`import` not `require`).

```js
import { EleventyEdge } from "./_generated/eleventy-edge.js";

export default async (request, context) => {
  try {
    let edge = new EleventyEdge("edge", {
      request,
      context,

      // default is [], add more keys to opt-in e.g. ["appearance", "username"]
      cookies: [],
    });

    edge.config(eleventyConfig => {
      // Run some more Edge-specific configuration
      // e.g. Add a sample filter
      eleventyConfig.addFilter("json", obj => JSON.stringify(obj, null, 2));
    });

    return await edge.handleResponse();
  } catch(e) {
    console.log( "ERROR", { e } );
    return context.next(e);
  }
};
```

#### Read more about Netlify’s Edge Functions

* {% indieweblink "Netlify Docs: Edge Functions overview", "https://docs.netlify.com/netlify-labs/experimental-features/edge-functions/" %}
* {% indieweblink "Netlify Edge Functions on Deno Deploy", "https://deno.com/blog/netlify-edge-functions-on-deno-deploy" %}
* {% indieweblink "Netlify Edge Functions: A new serverless runtime powered by Deno", "https://www.netlify.com/blog/announcing-serverless-compute-with-edge-functions" %}


### 4. `netlify.toml`

<details><summary>If you don’t already have a <code>netlify.toml</code>, expand this to view a sample starter.</summary>

```toml
[dev]
framework = "#static"
command = "npx @11ty/eleventy --quiet --watch"

[build]
command = "npx @11ty/eleventy"
publish = "_site"
```

</details>

Add this to your `netlify.toml` file. `eleventy-edge` points to the file you created above at `./netlify/edge-functions/eleventy-edge.js`.

```toml
[[edge_functions]]
function = "eleventy-edge"
path = "/*"
```

Feel free to change `path = "/*"` to something more granular!

### 5. Make your content template

Here we are making a simple `index.liquid` file. We can use the `{% raw %}{% edge %}{% endraw %}` shortcode to run the Liquid template syntax inside on the Edge server.

{% raw %}
```liquid
The content outside of the `edge` shortcode is generated with the Build.

{% edge %}
The content inside of the `edge` shortcode is generated on the Edge.

<pre>
{{ eleventy | json }}
</pre>
{% endedge %}
```
{% endraw %}

### 6. Run your local server

```
npx netlify dev
```

Navigation to `index.liquid` by going to `http://localhost:8888/` in your browser. (Double check your console output to make sure the port is `8888`).

## Frequently Asked Questions

### Limitations

_In progress._

### How does it compare to Serverless?

They can be used together! _In progress._