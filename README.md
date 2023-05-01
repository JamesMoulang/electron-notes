What do we need to do to create and distribute an electron app?

1. Actually write the thing
2. Bundle the thing into static assets
3. Create a simple electron application that serves those static assets
4. Package the electron application
5. Add native functionality to the electron app (if needed)
6. Allow the electron app's renderer process to talk to this native functionality

# Actually writing the thing
Use [vite](https://vitejs.dev/guide/) to run a dev server.

```
pnpm create vite
```

The basic structure of the project looks like this:

```
├── package.json
├── index.html (this is what is served when you connect to the dev server in a browser)
├── dist (this is created when it's bunled into static assets)
├── public
    ├── images
    └── audio
├── node_modules
└── src (contains project javascript files)
```

Run using:

```
pnpm run dev
```

# Bundling the thing into static assets

```
pnpm vite build
```

This creates a folder called dist which has a structure that looks like this:
```
├── index.html (this is what is served when you connect to the dev server in a browser)
├── assets
    └── index-48f348fc.js (bundled javascript files)
├── images
└── audio
```

In order for images/audio to be included in dist they must be imported from somewhere within the js. For example:
```javascript
import img_tick from '/images/tick.png';
const img = new Image();
img.src = img_tick;
```

What we have at this point is a folder of static assets ready to be served by an electron app.

If you try to run this in a browser, it will kind of work, but complains about CORS. You can run a preview with:

```
pnpm exec vite preview
```

# Creating an electron app to serve static assets

I'll create the skeleton of the electron app by using [electron-quick-start](https://github.com/electron/electron-quick-start)

```bash
# Clone this repository
git clone https://github.com/electron/electron-quick-start
# Go into the repository
cd electron-quick-start
# Install dependencies
pnpm i
# Run the app
pnpm run start
```

A basic Electron application needs just these files:

- `package.json` - Points to the app's main file and lists its details and dependencies.
- `main.js` - Starts the app and creates a browser window to render HTML. This is the app's **main process**.
- `index.html` - A web page to render. This is the app's **renderer process**.
- `preload.js` - A content script that runs before the renderer process loads.

Copy in assets, images, audio, and index.html to the electron app. (🔧 This is something I'd like to automate in the future).

In order to get this work, we have to run

```
pnpm vite build --base=./
```

to rewrite asset paths. Why? Because '/' in electron refers to the root directory of the file system, not the app's folder.

Let's edit the "build" command in package.json so we can just run:

```
pnpm exec build
```

# Packaging the electron app

We have a few options here. What's the best?

- electron-forge
- electron-builder
- electron-packager (which forge and builder both use)

Let's try electron-packager

```
pnpm add -D electron-packager
pnpm exec electron-packager x
```

Seems like pnpm doesn't play well with this.
