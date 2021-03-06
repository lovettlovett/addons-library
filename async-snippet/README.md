## Async Snippet Installation 
The scripts below illustrate a simple flicker management mechanism for asynchronous installs of Optimizely X Web. The mechanism works by masking certain elements until all syncronous Optimizely variation code has been executed, preventing the "flicker" of original content as the page is loading.

### Option 1. Masking the entire `<body>`
> _no configuration necessary_

This variation will set `opacity:0` to the <body> prior to any elements becoming visible. Once Optimizely syncronous changes have been applied, that body will be unhidden.

#### Approach
* Within the `<head>`, add CSSRule to dynamic CSSStyleSheet that says body `{opacity: 0}`
* Wait until Optimizely synchronous changes have been applied (`lifecycle.activated`)
* Disable dynamic CSSStyleSheet, unhiding body.

sample code:
```html
<!DOCTYPE html>
<html>
<head>
<script type="text/javascript">
  var maskTimeout          = 3000,
      syncChangesApplied   = false;

  /**
  * Manages CSSStyleSheet actions
  * Handles adding and removing rules from our "masking" sheet
  */
  var cssRuleManager = {
    sheet: (function() {
      var style = document.createElement("style");
      style.appendChild(document.createTextNode(""));
      document.head.appendChild(style);
      return style.sheet;
    })(),
    addCSSRule: function(selector, rules) {
      if ("insertRule" in this.sheet) {
        this.sheet.insertRule(selector + "{" + rules + "}", 0);
      } else if ("addRule" in this.sheet) {
        this.sheet.addRule(selector, rules, 0);
      }
    }
  }  

  /**
  * Fired in the first call of Optimizely `action.applied`
  * Disables "masking" stylesheet
  */
  var removeMask = function() {
    if(syncChangesApplied) return;
    cssRuleManager.sheet.disabled = true;
  }

  // Mask <body> immediately
  cssRuleManager.addCSSRule('body', 'opacity:0');

  /**
  * Listen for first sync change applied
  * and unmask nodes
  */
  window.optimizely = window.optimizely || [];
  window.optimizely.push({
    type: "addListener",
    filter: {
      type: "lifecycle",
      name: "campaignDecided"
    },
    "handler": removeMask
  }); 

  setTimeout(removeMask, maskTimeout);
</script>    
<script type="text/javascript" src="https://cdn.optimizely.com/js/PROJECTID.js" async></script>
</head>
<body>

    <h1>Async Snippet</h1>

</body>
</html>
```
---

### Option 2. Masking individual elements
> _configuration required_

This variation will set `opacity:0` to the individual elements that are being manipulated as part of the variation treatment. Once Optimizely syncronous changes have been applied, the hidden elements will reappear. 

#### Approach
* Supply an Array of element selectors to become hidden
* For each selector supplied, add CSSRule to dynamic CSSStyleSheet that says SELECTOR `{opacity: 0}`
* Wait until Optimizely synchronous changes have been applied (`lifecycle.activated`)
* Disable dynamic CSSStyleSheet, unhiding body.

sample code

```html
<!DOCTYPE html>
<html>
<head>
<script type="text/javascript">
  var maskTimeout          = 3000,
      hideElementSelectors = ['h1', 'p:nth-of-type(2)'],
      syncChangesApplied   = false;      

  /**
  * Manages CSSStyleSheet actions
  * Handles adding and removing rules from our "masking" sheet
  */
  var cssRuleManager = {
    sheet: (function() {
      var style = document.createElement("style");
      style.appendChild(document.createTextNode(""));
      document.head.appendChild(style);
      return style.sheet;
    })(),
    addCSSRule: function(selector, rules) {
      if ("insertRule" in this.sheet) {
        this.sheet.insertRule(selector + "{" + rules + "}", 0);
      } else if ("addRule" in this.sheet) {
        this.sheet.addRule(selector, rules, 0);
      }
    }
  }  

  /**
  * Fired in the first call of Optimizely `action.applied`
  * Disables "masking" stylesheet
  */
  var removeMask = function() {
    // its adequate to remove mask class once on the first sync change
    if(syncChangesApplied) return;
    cssRuleManager.sheet.disabled = true;
  }

  /**
  * Creates all CSSRules based on supplied list of selectors to hide
  * Binds Optimizely `action.applied` to unhide nodes by removing CSSRules
  */
  hideElementSelectors.forEach(function(selector) {
    cssRuleManager.addCSSRule(selector, 'opacity:0');
  });

  /**
  * Listen for first sync change applied
  * and unmask nodes
  */
  window.optimizely = window.optimizely || [];
  window.optimizely.push({
    type: "addListener",
    filter: {
      type: "lifecycle",
      name: "campaignDecided"
    },
    "handler": removeMask
  });  

  setTimeout(removeMask, maskTimeout);
</script>    
<script type="text/javascript" src="https://cdn.optimizely.com/js/PROJECTID.js" async></script>
</head>
<body>

    <h1>Async Snippet</h1>

</body>
</html>
```
