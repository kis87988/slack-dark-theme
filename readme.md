# Slack Dark Theme

A darker, more contrasty, Slack theme.

# Preview

![Screenshot](https://dl.dropboxusercontent.com/s%2Fbfbspwfn9adpsfv%2FSlack%2520-%2520AWS%2520Community%2520Nordics%25202018-08-04%252001-13-44.png)

# Installing into Slack

Find your Slack's application directory.

* Windows: `%homepath%\AppData\Local\slack\`
* Mac: `/Applications/Slack.app/Contents/`
* Linux: `/usr/lib/slack/` (Debian-based)

Open up `resources/app.asar.unpacked/src/static/ssb-interop.js`

At the very bottom, add

```js
// First make sure the wrapper app is loaded
document.addEventListener("DOMContentLoaded", function() {

   // Then get its webviews
   let webviews = document.querySelectorAll(".TeamView webview");

   // Fetch our CSS in parallel ahead of time
   const cssPath = '<URL_TO_CSS_FILE>';
   let cssPromise = fetch(cssPath).then(response => response.text());

   let customCustomCSS = `
   :root {
         /* Your custom CSS overrides */
   }
   `

   // Insert a style tag into the wrapper view
   cssPromise.then(css => {
      let s = document.createElement('style');
      s.type = 'text/css';
      s.innerHTML = css + customCustomCSS;
      document.head.appendChild(s);
   });

   // Wait for each webview to load
   webviews.forEach(webview => {
      webview.addEventListener('ipc-message', message => {
         if (message.channel == 'didFinishLoading')
            // Finally add the CSS into the webview
            cssPromise.then(css => {
               let script = `
                     let s = document.createElement('style');
                     s.type = 'text/css';
                     s.id = 'slack-custom-css';
                     s.innerHTML = \`${css + customCustomCSS}\`;
                     document.head.appendChild(s);
                     `
               webview.executeJavaScript(script);
            })
      });
   });
});
```

Last bit is for you to host the CSS theme file, then use the URL to this file by replacing `<URL_TO_CSS_FILE>`.

You can add your own overrides to any CSS element used by slack or the provided CSS theme file.
Simply add your CSS below `/* Your custom CSS properties below */`.

Sidebar should be configured seperately and this can be done per slack server.
Follow instructions here: https://slackthemes.net/#/zebra

That's it! Restart Slack and see how well it works.

NB: You'll have to do this every time Slack updates.

# Development

`git clone` the project and `cd` into it.

Change the CSS URL to `const cssPath = 'http://localhost:8080/custom.css';`

Run a static webserver of some sort on port 8080:

```bash
npm install -g static
static .
```

In addition to running the required modifications, you will likely want to add auto-reloading:

```js
const cssPath = 'http://localhost:8080/custom.css';
const localCssPath = '/Users/bryankeller/Code/slack-black-theme/dark.css';

window.reloadCss = function() {
   const webviews = document.querySelectorAll(".TeamView webview");
   fetch(cssPath + '?zz=' + Date.now(), {cache: "no-store"}) // qs hack to prevent cache
      .then(response => response.text())
      .then(css => {
         console.log(css.slice(0,50));
         webviews.forEach(webview =>
            webview.executeJavaScript(`
               (function() {
                  let styleElement = document.querySelector('style#slack-custom-css');
                  styleElement.innerHTML = \`${css}\`;
               })();
            `)
         )
      });
};

fs.watchFile(localCssPath, reloadCss);
```

Instead of launching Slack normally, you'll need to enable developer mode to be able to inspect things.

* Mac: `export SLACK_DEVELOPER_MENU=true; open -a /Applications/Slack.app`

* Linux: `export SLACK_DEVELOPER_MENU=true && /usr/bin/slack`

* Windows (PowerShell):

```powershell
# Set environment variable
[System.Environment]::SetEnvironmentVariable('SLACK_DEVELOPER_MENU', 'true', 'Process')

# Launch Slack (replace x.y.z with the latest version)
& $env:LOCALAPPDATA\slack\app-x.y.z\slack.exe

# Open developer console by pressing: Ctrl + Alt + I
```

# License

Apache 2.0

# Credits

Original author: https://github.com/widget-/slack-black-theme

Main contributor for 3.2.0 update: https://github.com/Nockiro/slack-black-theme