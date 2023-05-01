What do we need to do to create and distribute an electron app?

1. Actually write the thing
2. Bundle the thing into static assets
3. Create a simple electron application that serves those static assets
4. Add any native functionality
5. Package the electron application

# Actually writing the thing
To do this, I'm using [vite](https://vitejs.dev/) to run a dev server.
[Guide](https://vitejs.dev/guide/) to setting up a new vite project.

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

Run the project using:

```
pnpm run dev
```

Vite doesn't have to use webpack to bundle all the files together - it uses ES modules. Luckily it's trivial to switch.

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

In order for images/audio to be included in dist they must be imported from somewhere within the js. Maybe there's another way to do this, but I haven't figured it out yet. For me this is fine as all assets are loaded in js. And dist will only include the assets I actually load. I don't need to keep two asset folders up to date.

Anyway - what we have at this point is a folder of static assets ready to be served by an electron app.

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

So now let's take what we built with vite and transfer it into the app (🔧 This is something I'd like to automate in the future).

What does this mean? Copy in assets, images, audio, and index.html to the electron app.
It didn't work. Why? Because in index.html, we're accessing js from:

```
/index-48f348fc.js
```

If I change to

```
index-48f348fc.js
```

it does work, but I'd rather not. The question is: what does '/' refer to in an electron app?
Based on some [googling](https://stackoverflow.com/questions/38889682/electron-looking-for-resources-in-root-folder) electron is using the root of the filesystem here. This is kind of an issue. You can see the electron-quick-start index.html loads from './'

```html
<script src="./renderer.js"></script>
```

When vite builds, it always tries to reference '/assets/index.js'. Either we need to change what vite does, or we need to change what electron considers the root directory.

So if we run
```
pnpm vite build --base=./
```

then vite will instead refer to ./index-48f348fc.js

And it works! Assets and all. Nice one vite.
