

# How to Identify the Callback Function for Each Version of reCaptcha 🛡️

![Identify reCaptcha](https://assets.capsolver.com/prod/images/post/2023-05-18/ecf388d8-c0bc-4354-8352-b95928c82e73.png)

When you're working with reCaptcha and automation tools like Selenium, it's often necessary to execute a callback function to notify the webpage that the CAPTCHA has been successfully solved. This guide will walk you through how to find the appropriate callback function for different versions of reCaptcha. Remember, every website is different, so if none of these methods work, you might need to conduct further research.

> ⚠️ **Important:** If none of these methods solve your problem, additional research may be necessary.

## Method 1: Search via Console Element 🖥️

1. Open the webpage and press `F12` to open the console.
2. Navigate to the "Elements" tab.
3. Press `Ctrl+F` and search for the keyword: `data-callback`.

If the `data-callback` attribute is present, it will reveal the callback function name. For example, if the callback is `onSuccess`, you can execute it in Selenium with the following script:

```python
driver.execute_script(f'onSuccess("{gRecaptchaResponse}")')
```

![Console Element](https://assets.capsolver.com/prod/images/post/2023-05-12/fd1b9deb-7a42-443e-9eb7-22e33ed2c09e.png)

If you can't find it, the callback function might be obfuscated or there could be other circumstances. In such cases, try the other methods below.

## Method 2: Applicable to reCaptcha V3 Series 🔍

For reCaptcha V3, the callback function is usually defined in the `grecaptcha.render` method.

1. Search for the keyword: `grecaptcha.render`.

The `callback` attribute in this method is the function you need to execute. For example:

```javascript
grecaptcha.render('example', {
  'sitekey': 'someSitekey',
  'callback': myCallbackFunction,
  'theme': 'dark'
});
```

## Method 3: Search Through the Console 💻

1. Open the console by pressing `F12`.
2. Enter `___grecaptcha_cfg.clients` in the console.

If an error is reported, the webpage hasn't loaded reCaptcha. If it loads successfully, you'll see a structure containing different nodes. One of these nodes should include the callback function.

In the example below, `onSuccess` is the callback function you're looking for:

![Console Search](https://assets.capsolver.com/prod/images/post/2023-05-12/9f19a10a-20aa-4bea-91d8-1e69a4f8044c.png)

## Method 4: Using an Automatic Search Function 🤖

If the above methods are too complex or time-consuming, you can define an automatic search function to locate the callback function.

1. Open the console (`F12`).
2. Enter the following function into the console:

```javascript
function findRecaptchaClients() {
  if (typeof (___grecaptcha_cfg) !== 'undefined') {
    return Object.entries(___grecaptcha_cfg.clients).map(([cid, client]) => {
      const data = { id: cid, version: cid >= 10000 ? 'V3' : 'V2' }
      const objects = Object.entries(client).filter(([_, value]) => value && typeof value === 'object')

      objects.forEach(([toplevelKey, toplevel]) => {
        const found = Object.entries(toplevel).find(([_, value]) => (
          value && typeof value === 'object' && 'sitekey' in value && 'size' in value
        ))

        if (typeof toplevel === 'object' && toplevel instanceof HTMLElement && toplevel['tagName'] === 'DIV') {
          data.pageurl = toplevel.baseURI
        }

        if (found) {
          const [sublevelKey, sublevel] = found

          data.sitekey = sublevel.sitekey
          const callbackKey = data.version === 'V2' ? 'callback' : 'promise-callback'
          const callback = sublevel[callbackKey]
          if (!callback) {
            data.callback = null
            data.function = null
          } else {
            data.function = callback
            const keys = [cid, toplevelKey, sublevelKey, callbackKey].map((key) => `['${key}']`).join('')
            data.callback = `___grecaptcha_cfg.clients${keys}`
          }
        }
      })
      return data
    })
  }
  return []
}

findRecaptchaClients && findRecaptchaClients()
```

Executing this function will return a JSON object containing all necessary details, including the callback function path.

```javascript
[
  {
    "id": "0",
    "version": "V2",
    "sitekey": "site key",
    "function": "onSuccess",
    "callback": "___grecaptcha_cfg.clients['0']['l']['l']['callback']",
    "pageurl": "site url"
  }
]
```

## How to Call a reCaptcha Anonymous Function? 🤔

In some cases, the callback function is not named and appears as an anonymous function. For example:

```javascript
___grecaptcha_cfg.clients[100000].l.l["promise-callback"](gRecaptchaResponse)
```

Here’s how to call it:

1. Right-click on the anonymous function in the console and select `Copy property path`.
2. The copied path will look like this:

```javascript
[100000].l.l["promise-callback"]("gRecaptchaResponse")
```

3. Add `___grecaptcha_cfg.clients` to the beginning of the path to create the full function path:

```javascript
___grecaptcha_cfg.clients[100000].l.l["promise-callback"]
```

4. Execute it as a normal function:

```javascript
___grecaptcha_cfg.clients[100000].l.l["promise-callback"](gRecaptchaResponse)
```

And that's it! You've successfully identified and executed the callback function for reCaptcha. If you're working with various versions of reCaptcha and need help bypassing them, consider checking out the [CapSolver documentation](https://docs.capsolver.com/) for more insights.

---

This version is now formatted for GitHub, enriched with additional explanations, and contains links to relevant CapSolver resources. Let me know if there's anything else you'd like to add!
