# ReactJS-on-SharePoint-Online

- If you want to use ReactJS in SharePoint Online, this steps can help you setup your development environment.
- This does not use SPFx. This development setup is designed to be used if you are using a content editor in SPO to develop your site.
- I documented this as I had a very hard time making ReactJS development work with SharePoint Online and I can't really find a forum / documentation that discusses this kind of setup.

## PC Requirements

- Node.js and npm
- Visual Studio Code
- Command Prompt / Windows Powershell / Cmder , etc.

## Install Facebook's [create-react-app](https://github.com/facebook/create-react-app)

```
 npm i create-react-app
```

## Open your terminal, go to the directory where you want to create your project

```
 cd [foldername]
```

## Create your project using create-react-app

```
 create-react-app [projectname]

 example:
 create-react-app myproject
```

## Open your project in Visual studio code by running the following in your terminal

```
 cd myproject
 code .
```

## Install `node-sass` to enable SASS in your project.

Run:

```
 npm install node-sass --save-dev
```

You should change your css extensions to scss!

## Install [SPA](https://github.com/s-KaiNet/spsave) via npm

-- This package will allow us to upload/save files/folder to your SharePoint Online site via node.js
-- Install only as a `devDependency` (--save-dev / -D)

```
 npm install spsave --save-dev
```

### Create `sts.js` file in your parent project folder and copy the codes below (replace `siteUrl`, `username`, `password`, `folder`)

```javascript
/*
 * siteUrl - SharePoint Web absolute URL
 * username - MFCGD username
 * password - your domain password ( )
 * folder - folder path where you would upload the build assets
 */

var spsave = require("spsave").spsave;

var coreOptions = {
  siteUrl: "https://mfc.sharepoint.com/sites/AnnualCompCycle-UAT/zeke/",
  notification: true,
  checkin: true,
  checkinType: 1
};

var creds = {
  username: "John_E_Sebulino@mfcgd.com",
  password: "sharepoint password here"
};

var fileOptions = {
  folder: "folder to upload the file",
  glob: "build/**/*.*",
  base: "build"
};

spsave(coreOptions, creds, fileOptions)
  .then(function() {
    console.log("saved");
  })
  .catch(function(err) {
    console.log(err);
  });
```

## Install `sp-rest-proxy` and `concurrently`

Run:

```
  npm install sp-rest-proxy concurrently --save-dev
```

### Add the following to the `scripts` object of your project `package.json`

```JSON
    "proxy": "node ./api-server.js",
    "startServers": "concurrently --kill-others \"npm run proxy\" \"npm run start\""
```

### Add API proxy setting into `package.json`

```JSON
    "proxy": "http://localhost:8081"
     /*
        This is the address which corresponds to sp-rest-proxy startup settings.
         Proxy setting is a webpack serve feature which transfers localhost request to the sp-rest-proxy
     */
```

- Your `package.json` file should look something like on the example after this step.

### Create proxy server script in your parent project folder

```JS
    /*
    * The name [api-server.js] is user defined,
    * the below command should be executed in your terminal if you have `touch-cli` installed on your machine.
    * if you don't have it, you can install it or create the file manually in your project folder.
    */
    touch api-server.js

    /*
     * The api-server.js should contain the following codes
    */
    const RestProxy = require('sp-rest-proxy');

    const settings = {
        port: 8081 /* You may select your own port here */
    };

    const restProxy = new RestProxy(settings);
    restProxy.serve();

```

### Configure `sp-rest-proxy` :

Run:

```
 npm run proxy
```

-Connection parameters will be prompted (`username`, `password`, `SharePoint Online WebAbsolute URL`)
-After completing the prompt, it will create a folder and file (in your project) "config/private.json". This file stores the basic configuration settings of sp-proxy.
-Check if your credentials are correcy by navigating to `http://localhost:8081` and executiny any REST CALL
-Stop `sp-rest-proxy` , `Ctrl + C` on terminal

### Start local dev serve:

Run:

```
npm run startServers
```

Now when both servers have been started your React app can request for SharePoint API as if it were already deployed to SharePoint page, WebPack proxies local API requests to sp-rest-proxy and then requests to real SharePoint instance.

E.g., if open http://localhost:3000 in a browser and run:

```JS
fetch(`/_api/web`, {
    accept: 'application/json;odata=verbose',
})
  .then(r => r.json())
  .then(console.log)
  .catch(console.log)

```

## Navigate to your project folder (inside VSC) and open `package.json` file.

### Add the following line to the JSON:

```
"homepage": "SharePoint site assets folder"

example : "homepage": "/sites/AnnualCompCycle-UAT/developerAssets"
```

and

Add the following to `package.json` inside `scripts` object

```
"spo-build": "npm run build && node ./sts.js"
```

### Your `package.json` should look something like this

```JSON
{
  "name": "react-api",
  "version": "0.1.0",
  "private": true,
  "homepage": "/sites/AnnualCompCycle-UAT/zeke/zekewebassets/zekeBuild",
  "dependencies": {
    "react": "^16.8.6",
    "react-dom": "^16.8.6",
    "react-router-dom": "^5.0.0",
    "react-scripts": "3.0.0",
    "react-transition-group": "^1.2.1"
  },
  "proxy": "http://localhost:8081",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "spo-build": "npm run build && node ./sts.js",
    "proxy": "node ./api-server.js",
    "startServers": "concurrently --kill-others \"npm run proxy\" \"npm run start\""
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "devDependencies": {
    "spsave": "^3.1.5"
  }
}
```

## Try to build your project by running `npm run spo-build`

- This will build your project files in the background (js, css, html, images will be bundled)
- This will also run the sts.js file that we created earlier to upload the build files in your SharePoint Online Site
- If you get any error from this point, it is more likely that you have missed a step in the above instructions

## In your SharePoint Online Site, create a SharePoint page , Insert a content editor and link the index.html file that will be find in the build files uploaded to the folder in your SPO site

If you are able to view React's default content ( The Spinning React Icon) then you have done all steps correctly.

## Workflow

- Develop project locally
- Build the project by running `npm spo-build` to see it online

### That's it. This should help you get started with your Project if you're working with SharePoint Online and you want to use ReactJS :)

## Special Thanks To :

- [Andrew Kolyatkov](https://github.com/koltyakov) - blog @ [Getting started with React local development for SharePoint with sp-rest-proxy](http://blog.arvosys.com/2017/10/29/getting-started-with-react-local-development-for-sharepoint-with-sp-rest-proxy/index.html)
- David Petersen - blog @ [Developing ReactJS Single Page Apps for SharePoint](http://whatsthesharepoint.com/2016/11/developing-reactjs-single-page-apps-for-sharepoint/)
