---
title: Debug Extensions in Tableau Desktop 
layout: docs
---

A dashboard extension embeds a web page and runs a Chromium-based browser inside of Tableau Desktop. Fortunately, you can debug this embedded web browser using the remote debugging abilities built into Chromium.

Debugging an extension involves starting Tableau with a command option to enable remote debugging (over a debug port). You can then run a specific version of the Chromium debugger and connect to Tableau Desktop using the debug port.

<div class="alert alert-info"><b>Note</b> For Tableau Server or Tableau Online, see <a href= "/extensions-api/docs/trex_debug_server.html">Debugging Extensions in Tableau Server and Tableau Online.</a></div>

---
**In this section**

* TOC
{:toc}


---

## Download the Chromium Browser

To debug your extension, you'll need to use a Chromium-based browser (Chromium is the open-source version of Chrome). In some cases you could use Chrome, but because of version incompatibilities in the debugging protocol, we recommend using specific builds of Chromium, which matches the version of the browser running inside Tableau. Just download and unzip the file and run `chrome.exe` (Windows) or `chromium` (macOS).

Tableau Desktop version  |  Chromium version  | Chrome version
|----|----|----|
2018.2, 2018.3 | 47.0.2526.0 |  Not available
2019.1 and later | 79.0.3945.0  | Chrome version 79 or earlier.


**Chromium downloads for debugging Tableau 2018.2, 2018.3**

* [Chromium for Windows (47.0.2526.0)](https://www.googleapis.com/download/storage/v1/b/chromium-browser-snapshots/o/Win%2F352221%2Fchrome-win32.zip?generation=1443839123039000&alt=media)

* [Chromium for macOS (47.0.2526.0)](https://www.googleapis.com/download/storage/v1/b/chromium-browser-snapshots/o/Mac%2F352221%2Fchrome-mac.zip?generation=1443838516381000&alt=media)


**Chromium downloads for debugging Tableau 2019.1 and later**

* [Chromium for Windows (`chrome-win.zip`) (79.0.3945.0)](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/706915/)

* [Chromium for macOS (`chrome-mac.zip`) (79.0.3945.0)](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Mac/706915/)


<div class="alert alert-info"><b>Note </b> If you are using Tableau 2019.1 or later, you can debug extensions in Tableau Desktop using certain versions of Chrome (versions prior to 80). Currently, you can't use Chrome version 80 (or later) for debugging your extension.</div>


---


## Start Tableau Desktop and enable debugging (Windows)


1. Exit Tableau if it is already running on your computer. 
2. Open a Command Prompt window.
2. Start Tableau using the following command.
<br/>Replace `<version>` with the version of Tableau you are using (for example, `Tableau 2018.3`).

```
"C:\Program Files\Tableau\Tableau <version>\bin\tableau.exe" --remote-debugging-port=8696
```

This command enables remote debugging of extensions for this session of Tableau. 

**Note:** The remote debugging port (for example, `8696`) must match the port address you use with Chromium for debugging. This is *not* the HTTP port that you are using to host your extension, the port that is specified in the manifest file (`.trex`).

---

#### Optional - Create a Tableau shortcut for debugging (Windows)

For convenience, you could also create a shortcut for Tableau and add the remote debugging option to Properties dialog box.

1. Select Tableau from the **Start** menu, and then right-click and select **Properties** or select **Open file location**.<br/>
If you open the file location, you can create a new shortcut to `Tableau.exe` (call it something like *Tableau - Debug* ). Right click the shortcut to open the Properties dialog box.
3. In the Properties dialog box, append `-remote-debugging-port=8696` at the end of the command in the **Target** text box. The debugging option goes after the closing quotation mark for `"Tableau.exe"`.

    ![Debug Shortcut]({{site.baseurl}}/assets/Tableau_shortcut_debug.png){:height="40%" width="40%"}

---
## Start Tableau Desktop and enable debugging (macOS)


1. Open a Terminal window.
2. Start Tableau using the following command.
<br/> Replace `<version>` with the version of Tableau you are using (for example,`2018.3.app`).

   ```
   open /Applications/Tableau\ Desktop\ <version>.app --args --remote-debugging-port=8696

   ```

This command enables remote debugging of extensions for this session of Tableau.

**Note:** The remote debugging port (for example, `8696`) must match the port address you use with Chromium for debugging. This is *not* the HTTP port that you are using to host your extension, the port that is specified in the manifest file (`.trex`).


---


## Debugging Tableau Desktop using Chrome/Chromium

After you install the Chromium browser and enable debugging in Tableau, you can start debugging your extension.

1. Open the dashboard with the extension you want to debug in Tableau. 
2. Start Chromium and set the URL to [`http://localhost:8696`](http://localhost:8696)  
   This will bring up the page selector UI. The port (for example, `8696`) must match the port address you specified as the remote debugging port when you started Tableau.  
3. Pick the extension you want to debug from this page, and debug just like you would any other web application (set breakpoints, step through the code, and examine variables).<br/>
Note that the name of the extension is based on the title of the web page that was loaded for the extension. If the extension hasn't loaded yet the name might be blank. See [Debugging loading and initialization issues](#debugging-loading-and-initialization-issues).


    ![Remote Debugging]({{site.baseurl}}/assets/remote_debugging.gif)

Note that you can only debug one extension, or instance of an extension, at a time. However, you can open a new Chromium tab or window to debug additional extensions.

---

## Debugging loading and initialization issues

It can be difficult to hit breakpoints that occur during the loading of your page because of the remote debugging process. To help with this, you can select a menu option that causes your extension to wait to load until you trigger it to proceed.

1. Select the extension in the dashboard and select **Debug Options** > **Pause Before Loading** from the shortcut menu.
2. Reload your extension. Select **Reload** from the shortcut menu.
3. In Chromium, go to the debugging homepage ([http://localhost:8696](http://localhost:8696)). 
4. Click the *second item* listed to attach to the browser instance of your extension.<br/>
**Note:** When you click the list item, it will be completely blank, but it is really there. The cursor changes so you can select it.
5. Click the **Sources** tab in Chromium, under **Event Listener Breakpoints**, click **Script** and enable the **Script First Statement** breakpoint. You just have to do this one time.
6. In Tableau Desktop, click the extension zone to load your page.<br/>
The debugger will pause each time the first statement of a script runs, allowing you to debug the startup process.
7. To get to your JavaScript code, click **Continue** several times. After your JavaScript is loaded, you can set a breakpoint in your startup code. 


![Startup Debugging]({{site.baseurl}}/assets/onload_debugging.gif)

---

## Known issues with debugging extensions in Tableau Desktop

---


#### Reload closes debugging connection (2020.1 and earlier)


While you are debugging your extension, there are times you might want to reload or refresh your web page to execute and debug different parts of your code. However, when you click **Reload** from the shortcut menu to reload your extension, the remote debugger loses the connection with the extension. The reason for this is that **Reload** option tears down and re-creates the browser control, which means you'll need to establish a new debugging session.


Note that you do not need to close and reopen the Chromium browser every time you click **Reload**. You can start another debugging session by entering the URL (`http://localhost:8696`) in the Chromium address bar and selecting the extension from the page selector.

For more information, see [What Happens When You Reload an Extension]({{site.baseurl}}/docs/trex_reload.html).





