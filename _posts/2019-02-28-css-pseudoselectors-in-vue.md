---
layout: post
title: CSS pseudo selectors in Vue
comments: true
spotifyTrack: spotify:track:2q9QqI7jzetzaNbGdAGI74
---

**TL;DR:** Pseudo element rules of CSS aren't part of the DOM, and therefore they can't be altered using JavaScript methods. Here you find a [possible workaround](#solution) to alter them using Vue!
{:.message}

Few days ago I stumbled upon <a href="https://github.com/simonwhitaker/github-fork-ribbon-css" target="_blank">this</a> pure CSS implementation of a GitHub Ribbon, so, for fun, I decided to port the same as a Vue Single File Component.  

The paramount part of the porting was simple enough, indeed both the template and the component scoped style were easily setup. I managed to:  

- scaffold a single file component with <a href="https://github.com/team-innovation/vue-sfc-rollup" target="_blank">vue-sfc-rollup</a>;

- introduce the CSS rules as defined in <a href="https://github.com/simonwhitaker/github-fork-ribbon-css" target="_blank">github-fork-ribbon-css</a>;
  
- glue the parts together.  
 
However, while implementing a small customization, I figured out a problem: my idea was to introduce a property containing RGB values (e.g.: `#51b982`) to dynamically update the background color of the ribbon.

So far, so good. I thought to customize the `background-color` CSS property with a `v-bind:style` directive containing the dynamically computed RGB value:

```html
<template>
    <a href="#" :style="computedStyle">Ribbon text</a>
</template>

<script>
export default {
 props: {
    color: { 
      type: String,
      default: "#364a5e"
    }
},

computed: {
    computedStyle() { 
      return {
        'background-color': ${this.color}
      };
    }
}
}
</script>
```

These few lines of code seemed enough for the feature, but they're not working. What?!

The reason is because in the <a href='https://github.com/simonwhitaker/github-fork-ribbon-css/blob/4c0359eacb6f9b9d1631ce724b8756143864f710/gh-fork-ribbon.css#L56' target="_blank">original CSS used for the component</a>, the actual `background-color` of the ribbon has been introduced using a pseudo element selector, namely "`:before`". As far as I know, pseudo element rules of CSS aren't part of the DOM, and therefore they can't be altered using JavaScript methods.  
So, the previous approach didn't work.

### Solution
To solve the issue, I started googling for how to dynamically change `:before` CSS pseudo selectors using Vue.  Unluckly I didn't find any solution. So eventually I came with my own idea, that is to dynamically create a `<style>` DOM element containing a class overriding the `:before` pseudo element with the `background-color` received as component prop.
Basically what I have to do is to introduce a method that creates a templated CSS string filled with the prop value, then create a `<style>` element in the DOM and assign the string to its `innerHTML`. The method looks like the following:

```javascript
export default {
  methods: {
    applyCSS() {
      let cssRule = `#vue-ribbon:before {background-color: ${this.color}}`;
      let style = document.createElement("style");
      style.type = "text/css";
      //append the style node as child of current component
      this.$el.appendChild(style);
      style.innerHTML = cssRule;
    }
  }
}
```

Cool! Now, to evaluate the CSS code and apply the background color as expected, it is enough to invoke the method in the proper Vue lifecycle hooks, namely when the component is either mounted or before its updates. So let's define two lifecycle hooks invoking `applyCSS()`:

```javascript
mounted: function() {
    this.applyCSS();
},

beforeUpdate: function() {
    this.applyCSS();
}
```

Through this approach we are finally able to override the `:before` pseudo selector using a Vue property (namely `#vue-ribbon:before {background-color: ${this.color}}`)! However, `applyCSS` still has some flaws. Indeed, considering the method as defined above, each invocation will create a different `<style>` node in the DOM. Why instead don't we create it only once to avoid any DOM pollution? Here you go:

```javascript
applyCSS() {
      let nodeId = "vue-ribbon-bkg";
      let style = document.getElementById(nodeId);
      if (!style) {
        style = document.createElement("style");
        style.id = nodeId;
        style.type = "text/css";
        this.$el.appendChild(style);
      }
       let cssRule = `#vue-ribbon:before { background-color: ${this.color}}`;
       style.innerHTML = cssRule;
    }
```

This final version of the method checks if a `<style>` JavaScript node identified as "`vue-ribbon-bkg`" exists in the DOM. If the node does not exist, the method creates it. Otherwise, it simply updates the dynamically generated CSS rule injecting it in the `innerHTML` of the `<style>` node. :boom:

### Final thoughts

To wrap up, with the help of the method above I was able to write a fully customizable GitHub ribbon in Vue. Although this approach might look a little an anti-pattern because Vue best practices do not advice to modify the DOM in place, personally I think that for this small scoped components not containing too much logic and not interacting with global state of applications, a solution like this would be fine.

[Here](https://github.com/P3trur0/vue-ribbon/blob/master/src/Ribbon.vue) on GitHub you can find the complete source code of `vue-ribbon`. There you may notice that `applyCSS` method is slightly different from the snippets above, because I made it a little bit richer than the one introduced here.  