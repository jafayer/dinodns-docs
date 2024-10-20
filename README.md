# DinoDNS

Note: like [DinoDNS](https://github.com/jafayer/dinodns) itself, these docs are a work in progress. Their purpose is to describe the general conceptual overview of the framework. Separate API documentation will be made available soon.

These docs were generated with [Docusaurus](https://docusaurus.io).

### Installation

```
$ npm i
```

### Local Development

```
$ npm start
```

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

### Build

```
$ npm run build
```

This command generates static content into the `build` directory and can be served using any static contents hosting service.

### Deployment

Using SSH:

```
$ USE_SSH=true yarn deploy
```

Not using SSH:

```
$ GIT_USER=<Your GitHub username> yarn deploy
```

If you are using GitHub pages for hosting, this command is a convenient way to build the website and push to the `gh-pages` branch.
