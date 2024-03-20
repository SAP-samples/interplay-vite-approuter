# Interplay between Vite and SAP Approuter
This is a setup for a local development environment that uses the [SAP Approuter](https://www.npmjs.com/package/@sap/approuter) _in combination_ with the frontend-tooling [Vite](https://vitejs.dev).

The Approuter will handle the authorization and authentication details, while one can still leverage the speed and Hot Module Replacement (HMR) of Vite's development server (dev server) to develop the User Interface (UI) quickly e.g., with the [UI5 Web Components in React](https://sap.github.io/ui5-webcomponents-react/?path=/docs/getting-started--docs).

Say you prefer a standalone Approuter and you want to deploy the app later on to Business Technology Platform (BTP) in a Cloud Foundry Environment,
your local version of the app will have basically the same structure as your deployed version of the app.

Besides, it is beneficial to require authentication and authorization locally—especially if any backend data requires it anyway.

The basic idea is the following. Let's say you have started the app and authenticated successfully. The Approuter will then forward the
browsers incoming request for the UI to the running Vite dev server. The latter returns the needed files to display the UI back to the Approuter—that in 
turn returns them back to the browser client. In other words the Approuter acts as a reverse-proxy here. If you now make changes to your UI code—because
the HMR is still working—your changes will reflect immediately in the webpage shown to you in the browser. This makes for a nice development experience
overall, while still incorporating authentication and authorization at the same time.

## Requirements
For this setup Node version 18 was used, but any newer Node LTS version should work as well. 

## Folder Structure
The `ui/` folder with the UI was bootstrapped with Vite as the underlying frontend-tooling and its `react-ts` template for React and Typescript.

Inside the `router/` folder you will find everything related to the Approuter,
including the config applied during local development (see `router/dev`).

## Approuter Configuration
The configuration of the Approuter, which is going to be applied during local development, is stored in its own folder in `router/dev`.
Here, define a "dummy" destination for local use in `router/dev/default-env.json` that points to the Vite dev server, that is also running locally. In the same way, you can define one for any backend web server, e.g., a server for the Cloud Application Programming Model ([CAP](https://cap.cloud.sap/docs/)), that requires authentication to access it.
```json
{
    "destinations": [
        {
            "name": "api",
            "url": "http://localhost:4004",
            "forwardAuthToken": true
        },
        {
            "name": "vite-dev-server",
            "url": "http://localhost:5173"
        }
    ]
}
```

Next, tell the Approuter to forward any incoming requests to one of these two destinations. This is done in the `router/dev/xs-app.json`.
```json
{
    "welcomeFile": "/index.html",
    "routes": [
        {
            "source": "^/api(.*)$",
            "target": "$1",
            "destination": "api",
            "csrfProtection": false
        },
        {
            "source": "^/(.*)$",
            "target": "/$1",
            "destination": "vite-dev-server",
            "authenticationType": "none"
        }
    ]
}
```

## Vite Configuration
Change the port of Vite's web socket used for HMR, i.e. to _5174_ in our case, inside the `ui/vite.config.ts`. This way the web socket will
run on its own separate port. Otherwise, you will get an error at runtime and the HMR won't work.
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    hmr: {
      port: 5174,
    },
  },
  build: {
    outDir: "../router/dist",
    emptyOutDir: true,
  },
});
```

## Installation and Startup
If you do not already have TypeScript support installed, install _ts-node_ and _typescript_ globally with:
```
npm install --global typescript ts-node
```

Next, install the needed local packages by running the following command —while being in the root project folder:
```
npm run setup
```

Lastly, to start both the Vite dev server and the Approuter, run this command in the same folder:
```
npm run start
```

You should now be able to open up the application at http://localhost:5000 and see the UI there.

## Known Issues
No known issues

## How to obtain support
[Create an issue](https://github.com/SAP-samples/<repository-name>/issues) in this repository if you find a bug or have questions about the content.

## Contributing
If you wish to contribute code, offer fixes or improvements, please send a pull request. Due to legal reasons, contributors will be asked to accept a DCO when they create the first pull request to this project. This happens in an automated fashion during the submission process. SAP uses [the standard DCO text of the Linux Foundation](https://developercertificate.org/).

## License
Copyright (c) 2024 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSE) file.
