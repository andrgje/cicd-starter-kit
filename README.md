# CI/CD Starter Kit

## Necessary config files

### `public/staticwebapp.config.json`

```js
{
	"navigationFallback": {
		"rewrite": "/index.html",
		"exclude": ["/img/*.{png,jpg,gif,webp}", "/css/*"]
	},
	"mimeTypes": { ".json": "text/json" }
}
```

### Pipeline configuration (NODE)

```yml
env:
  NODE_VERSION: 18
```

## NodeJS Example

```yml
name: Build and deploy Node.js app to Azure Web App - nodejs-api

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js version
        uses: actions/setup-node@v3
        with:
          node-version: "20.x"

      - name: npm install, build, and test
        run: |
          npm install
          npm run build --if-present
          npm run test --if-present

      - name: migrate and generate prisma
        run: |
          npx prisma migrate dev --name "${{ github.sha }}"
          npx prisma generate

      - name: Zip artifact for deployment
        run: zip release.zip ./* -r

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: node-app
          path: release.zip

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: "Production"
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: Unzip artifact for deployment
        run: unzip release.zip

      - name: "Deploy to Azure Web App"
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: "nodejs-api"
          slot-name: "Production"
          publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE_0B2CA25C91644315ADB89F252659EB18 }}
          package: .
```
