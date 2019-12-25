The Electron Framework
Electron is an Open-Source Framework that allows you to build Cross-Platform Desktop Applications with Web Technologies (HTML, CSS, and JavaScript).
You have access to native components, plus the ability to configure automatic updates, crash reporting, debugging and profiling.
It uses NodeJS and Chromium to run the application. Therefore, web development skills are completely transferable to an Electron application.
All you need is a little HTML, CSS and JavaScript to get started.
Electron was originally built for Githubs’ Atom Editor
Electron Architecture
In Electron, there are two main processes: The main process and the renderer process. The GUI is created by the main process and involves the creation of web pages through the multi-processing capabilities of Chromium.
Each Chromium Web Process runs its’ own renderer process. The renderer instances, created as BrowserWindow instances, are isolated and only concerned with its own web page.
Native Resources can be handled through the NodeJS API which, unlike in web applications, is accessible to the application to create lower-level operation system interactions. You can call Native GUI API’s through communication with the main process using ipcRenderer and ipcMain to send and receive messages.
What Electron Forge Offers
Electron-Forge is a complete tool that handles the creation of distributables designed to work with multiple systems, in addition to installing and publishing to a server.
Plugins
Electron Forge’s plugins allow extensibility of the core Electron Forge functionality.
Plugin’s such as the Webpack Plugin plugin-webpack , allows the use of webpack to build the application, with the option to enable hot-reloading and advanced bundle configuration.
There is also the plugin-auto-unpack-natives plugin, which will automatically add native modules in the node_modules . This reduces disk consumption and loading times on users’ machines.
You can even write your own plugins
Templates
Electron Forge’s Templates allow you to integrate existing web technologies such as the Webpack template (which uses the webpack plugin) and the TypeScript template, which provides you with a basic project configuration in TypeScript. The TypeScript template provides you with the required tsconfig.json file to transpile TypeScript files into JavaScript.
The TypeScript-Webpack Plugin typescript-webpack combines the webpack template (using the webpack plugin) with the typescript template (using the typescript plugin).
This is the template that we will use.
Makers
Electron Forge’s Makers Tool takes your packaged application and generates platform-specific distributables such as .dmg (macOS) and .exe (Windows).
Each Maker can have a platform-specific maker config, specifying which platforms to run for.
Publishers
Electron Forge Publishers take the artifacts that you create with a Maker, then send to an update server (to use as updates) or to a service (such as an S3 Bucket) for distribution. You can configure Publishers with platform specific instructions.
The Publishers default to publish to ALL platforms so you don’t have to specify a platform if you want the default behaviour.
Let’s Start Building 🚀
Prerequisites
Must have NodeJS & TypeScript installed.
Must have a package manager, such as npm or yarn, installed.
I’ll be using Node version 10.15.3 .
Install Electron & TypeScript Globally
$ npm i typescript electron -g
Let’s create a project
We will be using the typescript-webpack template to give us the necessary files to start using TypeScript and Webpack together.
$ npx create-electron-app electron-demo --template=typescript-webpack

If all goes well
If this does not succeed, make sure you have installed everything in prerequisites.
The generated files
The baseline project has now been created using the template.
As you can see from the project structure, the essential files have been automatically created with the typescript + webpack template.
There are separate configuration files for the main and renderer process: webpack.main.config.js and webpack.renderer.config.js respectively.
The TypeScript files that have been included in the tsconfig.json file.
This file will handle transpiling TypeScript into JavaScript during the build process, and tslint.json is used for linting TypeScript code for errors & recommendations using tslint.
The template will also generate the types .d file, making use of the @electron-forge/plugin-typescript package to use types (TypeScript).
You’ll also see the index.ts , which is responsible for the main process (which is why I like to rename it to main.ts as I find it more intuitive, being in charge of the MAIN process).

Change directory (cd) into electron-demo
$ cd electron-demo
Let’s change the name of the file at src/index.ts
$ mv src/index.ts src/main.ts
Before we get to React, we need to check the initially generated files
The main.ts file (renamed from index.ts)

main.ts
As you can see, the main window is loaded from a URL that is dynamically created via webpack magic constant MAIN_WINDOW_WEBPACK_ENTRY . To use the magic constant, MAIN_WINDOW_WEBPACK_ENTRY is declared at the top (to also satisfy any errors).
A BrowserWindow instance is then created with a minimal configuration (just height and width specified).
If you wanted Node Integration in your renderer process, this is where you would include nodeIntegration: true into the BrowserWindow config.
The renderer.ts file
This is where our main application functionality will start from. At the moment it is pretty empty, but as you can see index.css is automatically imported (which demonstrates the purpose of this file, which is to render the web pages).
Note: Will will be changing this file to renderer.tsx later on when we start using React.

renderer.ts
The tsconfig.json file
This controls the compiler options for TypeScript transpilation.

tsconfig.json
The webpack files
The only real difference between the webpack.main.config.js and webpack.renderer.config.js files, is that the renderer config includes the styles, and the main config points to the src/main.ts as the entry file.
The webpack.rules.js file includes rules for shared resources such as the native modules, the ts-loader, and node-loader.
To be able to use React, we are going to modify most of these files.
Let’s get React
We will break this down into a few steps:
Install dependencies
Let our compiler know we are using React/JSX
Let webpack know we are using React/JSX
Add our Service Worker
Replace our entry renderer file with a .tsx file
Install Dependencies
$ npm i react react-dom react-router-dom react-scripts
Tell our compiler about JSX
We need to tell the TypeScript compiler to use React JSX, change our outDir to .webpack/main, which is where our built application will be generated (by webpack), and include some libs.
This is what our tsconfig.json file should look like.

Tell Webpack About JSX
We need to include .jsx and .tsx in the regex test in our webpack config.
The renderer config:

webpack.renderer.config.js
To enable hot reloading, install $ npm i babel-loader @hot-loader/react-dom react-hot-loader and include it as an alias for react-dom . Also you need to make sure you resolve (include the regex test for) .tsx and .jsx files in the main and renderer configs as shown.
You’ll also need to create a .babelrc file, including the react-hot-loader as a plugin.

.babelrc
The main config:

webpack.main.config.js
The shared webpack rules:
Let’s install our development dependencies that are used by this file.
$ npm i css-loader style-loader node-sass sass-loader babel-loader ts-loader node-loader --save-dev

webpack.rules.js
As you can see I have included SASS/SCSS (.scss) files to be built using sass-loader. If you want to use SCSS then just $ npm i sass-loader and make sure you have included test: /\.(scss|css)$/ as above.
Service Worker
Let’s just keep it simple for now and include the default service worker file that is normally generated with create-react-app .

serviceWorker.ts
Some file changes
To keep the global index.html and index.scss or index.css separate from the react components, we will create the directory public specifically for those files in the project root.
Inside src :
I also like to have the main.ts file in a main directory under src/main and all other renderer/react files under src/renderer . We will also create a components directory under src/renderer/components to hold our React components.
$ mkdir public && mkdir src/renderer && mkdir src/main
Then move the files into their new directories.
So the new structure should be…

src and public
The renderer.tsx file

renderer.tsx (previously renderer.ts)
The App.tsx file
Wrapping the root App.tsx file in hot() enables hot module reloading. This will ensure that changes in any of our source files will be updated.

Package.json
We need to change the main entry and the electron forge renderer config to point to the correct index.html and renderer.tsx file.
Main entry
"main": "./.webpack/main",
Electron Renderer Config
"renderer": {
  "config": "./webpack.renderer.config.js",
  "entryPoints": [
    {
      "html": "./public/index.html",
      "js": "./src/renderer/renderer.tsx",
      "name": "main_window"
    }
  ]
}
Make distributable files
Let’s make our application.
I’m going to create a .dmg file for MacOS (OSX) just to show how easy it is to create another distributable.
However, the .zip file that is created by default can run on any platform.
To get a .dmg of our application, I would add this to our package.json in the makers config.
{
  "name": "@electron-forge/maker-dmg",
  "config": {
    "background": "../something_random.png",
    "format": "ULFO"
  }
},
Now make
$ electron-forge make

Run your app ⚡️
$ electron-forge start
