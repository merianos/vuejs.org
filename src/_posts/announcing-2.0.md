---
title: Ανακοίνωση του Vue.js 2.0
date: 2016-04-27 13:33:00
---

Σήμερα είμαι ενθουσιασμένος που ανακοινώνω την πρώτη δημόσια προεπισκόπηση του Vue.js 2.0, που φέρνει μαζί της πολλές συναρπαστικές βελτιώσεις και νέα χαρακτηριστικά. Ας ρίξουμε δούμε λοιπόν τι έχει η νέα έκδοση!

<!-- more -->

## Ακόμα ποιο λεπτό, Ακόμα ποιο γρήγορο

Το Vue.js από την αρχή είναι προσανατολισμένο προς το να παραμείνει ελαφρύ και γρήγορο, αλλά η έκδοση 2.0 το πάει ακόμα ποιο μακριά. Το επίπεδο απεικόνισης είναι πλέον βασισμένο στην ελαφριά υλοποίηση του εικονικού-DOM (που βασίζετε στο [Snabbdom](https://github.com/paldepind/snabbdom)) που βελτιώνει την ταχύτητα αρχικής απεικόνισης και την κατανάλωση μνήμης μέχρι 2~4x στα περισσότερα πιθανά σενάρια (δείτε τις [αναφορές απόδοσης](https://github.com/vuejs/vue/tree/next/benchmarks)). Ο μεταγλωττιστής του πρότυπου-σε-εικονικό-DOM και το περιβάλλον χρόνου εκτέλεσης μπορούν να διαχωριστούν, έτσι μπορείτε να προ-μεταγλωττίσετε τα πρότυπα _(templates)_ και να δημοσιεύσετε την εφαρμογή σας μόνο με το περιβάλλον χρόνου εκτέλεσης, που είναι μικρότερο από 12KB όταν είναι ελαχιστοποιημένο και σερβίρετε με gzip (ως αναφορά, το React 15 είναι 44KB ελαχιστοποιημένο και σερβιρισμένο με gzip). Επίσης, ο μεταγλωττιστής λειτουργεί και στον περιηγητή διαδικτύου, που σημαίνει πως μπορείτε ακόμα να βάλετε μια ετικέτα script στον κώδικα σας και να ξεκινήσετε να γράφετε κώδικα, ακριβώς όπως και πριν. Ακόμα και με τον μεταγλωττιστή μέσα, η εφαρμογή σας παραμένει στα 17KB ελαχιστοποιημένη και σερβιρισμένη με gzip, που είναι ακόμα ελαφρύτερη από την τρέχουσα έκδοση 1.0.

## Δεν είναι το Εικονικό-DOM του μέσου όρου

Τώρα, μόνο το εικονικό-DOM ακούγετε τόσο βαρετό, επειδή υπάρχουν τόσες πολλές υλοποιήσεις διαθέσιμες, αλλά υπάρχει μια υλοποίηση που διαφέρει. Συνδυασμένη με την διαδραστικότητα του Vue, παρέχει εξ ορισμού βελτιστοποιημένη επαναλαμβανόμενη απεικόνιση, χωρίς να πρέπει να κάνετε το παραμικρό. Κάθε στοιχείο _(component)_ διατηρεί τον έλεγχο όλων των διαδραστικών εξαρτήσεων του κατά την απεικόνιση, έτσι τ σύστημα γνωρίζει με ακρίβεια πότε πρέπει να ξανά απεικονιστεί, και ποια στοιχεία να απεικονίσει ξανά. Δεν υπάρχει πλέον ανάγκη για `shouldComponentUpdate` ή για αμετάβλητες δομές δεδομένων - **απλά λειτουργεί**.

Επιπλέον, το Vue 2.0 εφαρμόζει κάποιες εξειδικευμένες βελτιστοποιήσεις κατά την φάση μεταγλώττισης πρότυπο-σε-εικονικό-DOM:

1. It detects static class names and attributes so that they are never diffed after the initial render.

2. It detects the maximum static sub trees (sub trees with no dynamic bindings) and hoist them out of the render function. So on each re-render, it directly reuses the exact same virtual nodes and skips the diffing.

These advanced optimizations can usually only be achieved via Babel plugins when using JSX, but with Vue 2.0 you can get them even using the in-browser compiler.

The new rendering system also allows you to disable reactive conversions by simply freezing your data and manually force updates, essentially giving you full control over the re-rendering process.

With these techniques combined, Vue 2.0 ensures blazing fast performance in every possible scenario while requiring minimal optimization efforts from the developer.

## Templates, JSX, or Hyperscript?

Developers tend to have strong opinions on templates vs. JSX. On the one hand, templates are closer to HTML - they map better to the semantic structure of your app and make it much easier to think visually about the design, layout and styling. On the other hand, templates are limited to the DSL while the programmatic nature of JSX/hyperscript provides the full expressive power of a turing-complete language.

Being a designer/developer hybrid, I prefer writing most of my interfaces in templates, but in certain cases I do miss the flexibility of JSX/hyperscript. An example would be writing a component that programmatically handles its children, something not feasible with just the template-based slot mechanism.

Well, why not have both? In Vue 2.0, you can keep using the familiar template syntax, or drop down to the virtual-DOM layer whenever you feel constrained by the template DSL. Instead of the `template` option, just replace it with a `render` function. You can even embed render functions in your templates using the special `<render>` tag! The best of both worlds, in the same framework.

## Streaming Server-side Rendering

With the migration to virtual-DOM, Vue 2.0 naturally supports server-side rendering with client-side hydration. One pain point of current mainstream server rendering implementations, such as React's, is that the rendering is synchronous so it can block the server's event loop if the app is complex. Synchronous server-side rendering may even adversely affect time-to-content on the client. Vue 2.0 provides built-in streaming server-side rendering, so that you can render your component, get a readable stream back and directly pipe it to the HTTP response. This ensures your server is responsive, and gets the rendered content to your users faster.

## Unlocking More Possibilities

With the new architecture, there are even more possibilities to explore - for example, rendering to native interfaces on mobile. Currently, we are exploring a port of Vue.js 2.0 that uses [weex](http://alibaba.github.io/weex/) as a native rendering backend, a project maintained by engineers at Alibaba Group, the biggest tech enterprise of China. It is also technically feasible to adapt Vue 2.0's virtual-DOM to run inside ReactNative. We are excited to see how it goes!

## Compatibility and What to Expect Next

Vue.js 2.0 is still in pre-alpha, but you can checkout the source code [here](https://github.com/vuejs/vue/tree/next/). Despite being a full rewrite, the API is largely compatible with 1.0 with the exception of some intentional deprecations. Check out [the same official examples written in 2.0](https://github.com/vuejs/vue/tree/next/examples) - you will see that not much has changed!

The feature deprecations are part of our continued effort to provide the simplest API possible for maximum developer productivity. You can check out a 1.0 vs. 2.0 feature comparison [here](https://github.com/vuejs/vue/wiki/2.0-features). This does mean that it will take some effort to migrate an existing app if you happen to use some of these deprecated features heavily, but we will provide detailed upgrade guides in the future.

There is still much work left to be done. We will be releasing the first alpha once we reach satisfactory test coverage, and we are aiming for beta by end of May / early June. In addition to more tests, we also need to update the supporting libraries (vue-router, Vuex, vue-loader, vueify...). Currently only Vuex works with 2.0 out of the box, but we will make sure that everything works smoothly together when 2.0 ships.

We are also not forgetting about 1.x! 1.1 will be released alongside 2.0 beta, with an LTS period of 6-month critical bug fixes and 9-month security updates. It will also ship with optional deprecation warnings to get you prepared for upgrading to 2.0. Stay tuned!
