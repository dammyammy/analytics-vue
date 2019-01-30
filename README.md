<div align="center">
  <img src="https://user-images.githubusercontent.com/16131737/51560489-81227900-1e39-11e9-9998-62431a719886.png"/>
</div>

This tutorial will help you start sending data from your Vue application to Segment and any of our destinations, using our <a href="https://segment.com/docs/sources/website/analytics.js/">Analytics.js library</a>. As soon as you're setup you'll be able to turn on any new destinations with the flip of a switch!

Want to try it for yourself? Scroll down to the <a href="#demo">demo section</a> and run the app!

Start sending data from any [source](https://segment.com/docs/guides/general/what-is-a-source) and see events live in the Segment **debugger**:

<div align="center">
  <img src="https://user-images.githubusercontent.com/16131737/52017555-87010580-249c-11e9-8848-896ac1f15daa.gif"/>
</div>
<br/>

Once you have data being sent to Segment, forward this data to any of our 250+ [destinations](https://segment.com/docs/guides/general/what-is-a-destination):

<div align="center">
  <img src="https://user-images.githubusercontent.com/16131737/52017558-88323280-249c-11e9-9895-ebcd3e37c040.gif"/>
</div>

# 🔌 Installation
## ✂️ Step 1: Copy the Snippet
Installing Segment is easy, just paste this snippet into the head of your site. When you paste it, you'll need to replace `YOUR_WRITE_KEY` with your Segment project's <b>Write Key</b>, which you can find in your project setup guide or settings:
```html
<script type="text/javascript">
  !function(){var analytics=window.analytics=window.analytics||[];if(!analytics.initialize)if(analytics.invoked)window.console&&console.error&&console.error("Segment snippet included twice.");else{analytics.invoked=!0;analytics.methods=["trackSubmit","trackClick","trackLink","trackForm","pageview","identify","reset","group","track","ready","alias","debug","page","once","off","on"];analytics.factory=function(t){return function(){var e=Array.prototype.slice.call(arguments);e.unshift(t);analytics.push(e);return analytics}};for(var t=0;t<analytics.methods.length;t++){var e=analytics.methods[t];analytics[e]=analytics.factory(e)}analytics.load=function(t,e){var n=document.createElement("script");n.type="text/javascript";n.async=!0;n.src="https://cdn.segment.com/analytics.js/v1/"+t+"/analytics.min.js";var a=document.getElementsByTagName("script")[0];a.parentNode.insertBefore(n,a);analytics._loadOptions=e};analytics.SNIPPET_VERSION="4.1.0";
  analytics.load("YOUR_WRITE_KEY");
  }}();
</script>
```
Now `window.analytics` is loaded and available to use throughout your app!

###  Single-Page Application
The `page` method lets you record page views on your website, along with optional information about the page being viewed. You can read more about in the [page reference](https://segment.com/docs/sources/website/analytics.js/#page). Clicking a link or a new tab will not reload the webpage in an SPA. Therefore, using `analytics.page()` in `index.html` is not ideal and we need to simulate a page load. However, we can achieve `page` calls with the use of [vue-router](https://router.vuejs.org) and Vue's lifecycle hooks.

If we separate our pages into their own components and allow the [`<router-view>`](https://router.vuejs.org/api/#router-view) component to handle when our pages render, then we can use `mounted` lifecycle hook to invoke our `page` calls:

```javascript
<template>
  <h1>
    Home page.
  </h1>
</template>

<script>
export default {
  name: 'HomePage',
  mounted () {
    window.analytics.page('Home')
  }
}
</script>
```

##  🔍 Step 2: Identify Users
The `identify` method is how you tell Segment who the current user is. It includes a unique User ID and any optional traits you know about them. You can read more about it in the <a href="https://segment.com/docs/sources/website/analytics.js/#identify">identify reference</a>.

<b>Note:</b> You won't need to call `identify` for anonymous visitors to your site. We'll automatically assign them an `anonymousId`, so just calling `page` and `track` will still work just fine without `identify`.

Here's what a basic call to `identify` might look like:

```javascript
window.analytics.identify('f4ca124298', {
  name: 'Michael Bolton',
  email: 'mbolton@initech.com'
});
```

That's identifying Michael by his unique User ID and labeling him with `name` and `email` traits.

### Event Handler
If you're using a form to handle user signups or logins, the `v-on:submit` handler is a great use-case to call `identify`:
```javascript
<template>
  <form v-on:submit="handleSubmit">
    <input name="name" type="text" v-model="name" />
    <input name="email" type="email" v-model="email" />
    <input type="submit" value="Submit" />
  </form>
</template>

<script>
export default {
  name: 'IdentifyForm',
  data: {
    name: '',
    email: ''
  },
  methods: {
    handleSubmit () {
      // Add your own unique ID here or we will automatically assign an anonymousID
      window.analytics.identify({
        name: this.name,
        email: this.email
      })
    }
  }
}
</script>
```

## ⏰ Step 3: Track Actions
The `track` method is how you tell Segment about which actions your users are performing on your site. Every action triggers what we call an "event", which can also have associated properties. It is important to figure out exactly what events you want to `track` instead of tracking anything and everything. You can read more about `track` in the <a href="https://segment.com/docs/sources/website/analytics.js/#track">track reference</a>.

Here's what a call to `track` might look like when a user bookmarks an article:

```javascript
window.analytics.track('Article Bookmarked', {
  title: 'Snow Fall',
  subtitle: 'The Avalanche at Tunnel Creek',
  author: 'John Branch'
});
```

That's telling us that your user just triggered the <b>Article Bookmarked</b> event and bookmarked the `Snow Fall` article authored by `John Branch`. Properties can be anything you want to associate to an event when it is tracked.

### Event Handler
[Event handlers](https://vuejs.org/v2/guide/events.html) are oftenly used to call `track`, such as: `v-on:click`, `v-on:submit`, `v-on:mouseover`, `etc.`:

```javascript
<template>
  <button v-on:click="trackEvent">
    {{ title }}
  </button>
</template>

<script>
export default {
  name: 'SignupButton',
  data: {
    return {
      title: 'Signup with Segment today!'
    }
  },
  methods: {
    trackEvent () {
      window.analytics.track('User Signup')
    }
  }
}
</script>
```

### Lifecycle Hook
[Lifecycle hooks](https://vuejs.org/v2/api/#Options-Lifecycle-Hooks) are also great for tracking particular events. For example, if you want to track components that are conditionally rendered from a parent component and that are outside the scope of a `page` call, then you can use `mounted` to trigger a `track` event:

```javascript
<template>
  <video autoplay>
    <source src="https://www.youtube.com/watch?v=dQw4w9WgXcQ" type="video/mp4">
  </video>
</template>

<script>
export default {
  name: 'VideoPlayer',
  mounted () {
    window.analytics.track('Video Played')
  }
}
</script>
```

### Transition
Using a [transition](https://vuejs.org/v2/guide/transitions.html) wrapper component to render a piece of UI? The available hooks (`beforeEnter`, `enter`, `enterCancelled`, `leave`, `etc.`) can be used to call `track` for particular events:

```javascript
<template>
  <div>
    <button v-on:click="show = !show">
      {{ title }}
    </button>
    <transition v-on:after-enter="afterEnter">
      <p v-if="show">
        {{ text }}
      </p>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'DestinationsToggle',
  data: {
    return {
      title: 'Toggle',
      text: 'Integrate with over 200+ destinations!'
    }
  },
  methods: {
    afterEnter (el) {
      window.analytics.track('Destinations Info Toggled')
    }
  }
}
</script>
```

### Error Boundary
A higher-order component to wrap around children components can be useful for catching errors. Usually when an error occurs, we will log the error with `track` and gracefully display the appropriate child component:

```javascript
<script>
import DefaultErrorComponent from './DefaultErrorComponent.vue'

export default {
  name: 'ErrorBoundary',
  props: {
    errorComponent: {
      type: Object,
      default: () => DefaultErrorComponent
    }
  },
  data: {
    return {
      error: false
    }
  },
  errorCaptured (err, vm, info) {
    this.error = true

    window.analytics.track('JavaScript Error', {
      error: err,
      errorInfo: info
    })
  },
  render (h) {
    return this.error ? h(this.errorComponent) : this.$slots.default[0]
  }
}
</script>
```

Once you've added a few track calls, **you're done**! You successfully installed `Analytics.js` tracking. Now you're ready to turn on any destination from our interface. 🎉

## 🎓 Advanced
### Typecheck
Typechecking with [`prop-types`](https://vuejs.org/v2/guide/components-props.html) can catch a lot of potential bugs and prevent handing down information in the wrong format. For example, enforcing a format for `user` related objects can help with data standardization. You can get creative with the traits you expect to be sent to Segment for `identify` and `track`:

```javascript
<script>
// Object with type declaration is only valid with Typescript
type IdentifyTraits = {
  name: string
  email: string
  isAuthorized: boolean
}

export default {
  name: 'User',
  props: {
    id: String,
    identifyTraits: Object as () => IdentifyTraits,
    trackTitle: String,
    trackTraits: Object
  },
  render () {
    return this.$slots.default[0]
  }
}
</script>
```

# 📺 <span name="demo">Demo</span>
1. Add your Segment <b>Write Key</b>, which you can find in your project setup guide or settings, to the snippet in <a href="https://github.com/segmentio/analytics-vue/blob/master/public/index.html#L11">index.html</a>:
```html
<script type="text/javascript">
  !function(){var analytics=window.analytics=window.analytics||[];if(!analytics.initialize)if(analytics.invoked)window.console&&console.error&&console.error("Segment snippet included twice.");else{analytics.invoked=!0;analytics.methods=["trackSubmit","trackClick","trackLink","trackForm","pageview","identify","reset","group","track","ready","alias","debug","page","once","off","on"];analytics.factory=function(t){return function(){var e=Array.prototype.slice.call(arguments);e.unshift(t);analytics.push(e);return analytics}};for(var t=0;t<analytics.methods.length;t++){var e=analytics.methods[t];analytics[e]=analytics.factory(e)}analytics.load=function(t,e){var n=document.createElement("script");n.type="text/javascript";n.async=!0;n.src="https://cdn.segment.com/analytics.js/v1/"+t+"/analytics.min.js";var a=document.getElementsByTagName("script")[0];a.parentNode.insertBefore(n,a);analytics._loadOptions=e};analytics.SNIPPET_VERSION="4.1.0";
  analytics.load("YOUR_WRITE_KEY");
  }}();
</script>
```

2. Install the dependencies and run the app:
```javascript
npm install
npm start
```

3. View the live events being triggered in your Segment dashboard debugger:
    + Page event: `Home`
    + Page event: `About`
    + Track event: `Learn Vue Link Clicked`

## 💾 Warehouses
A warehouse is a special subset of destinations where we load data in bulk at a regular intervals, inserting and updating events and objects while automatically adjusting their schema to fit the data you've sent to Segment. We do the heavy lifting of capturing, schematizing, and loading your user data into your data warehouse of choice.

Examples of data warehouses include Amazon Redshift, Google BigQuery, MySQL, and Postgres.

<div align="center">
  <img src="https://user-images.githubusercontent.com/16131737/52017561-8b2d2300-249c-11e9-9f59-efad34ce929c.gif"/>
</div>

# 🤔 What's Next?
Check out our full <a href="https://segment.com/docs/sources/website/analytics.js/">Analytics.js reference</a> to see what else is possible, or read about the <a href="https://segment.com/docs/sources/server/http/">Tracking API methods</a> to get a sense for the bigger picture.

If you have any questions, or see anywhere we can improve our documentation, <a href="https://segment.com/contact/">please let us know</a>!
