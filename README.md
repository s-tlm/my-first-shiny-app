# my-first-shiny-app

## Programmatic Deployment

The following uses the `rsconnect` package.

### Posit Connect

The following functions can be used.

- `deployDoc` - Deploy a single R Markdown, Quarto document or `.html` or `.pdf` file.
- `deployApp` - Deploy a *shiny* application, *RMarkdown* document, plumber API or HTML content to a server.
- `deployAPI` - Deploy an application consisting of plumber API routes.
- `deploySite` - Deploy an R Markdown or quarto website to a server.

> [!TIP]
> Example usage: `rsconnect::deployApp("./src", appName="my-first-shiny-app")`

#### 1. Build the Bundle

The bundle contains the application source code, data files and a manifest (JSON) file with the application metadata.

`rsconnect` infers some attributes based on the content being deployed.

1. `appMode` - based on the content being deployed.
2. `hasParameters` - whether the R Markdown deployment has parameters.

Next, `rsconnect` finds the relevant files for the application.

`appFiles` or `appFileManifest` can be passed as arguments to the `deployApp` function. Otherwise, `rsconnect` attempts to identify the files.

For Shiny applications and Plumber APIs, `rsconnect` will add all the files in the project directory and sub-directories to the bundle, except: `.Rproj` file, `/packrat` and `/rsconnect`.

> [!TIP]
> Use `rsconnect::listBundleFiles(appDir)` to see the identified dependencies.

After the target files are located, `rsconnect` will lint them. It tries to find any problems with the application that might prevent it from working post-deployment.

It checks the following.

1. Absolute paths.
2. Invalid relative paths.
3. Inconsistent capitalisation among paths (Posit Connect server paths are case sensitive).

`rsconnect` will now identify the packages needed by the application using `packrat`. If an `renv` environment is activated, `rsconnect` will pull dependencies from the `renv.lock` file. All packages are written to a `packrat` lock file.

Finally, `rsconnect` builds the `manifest.json` file containing the application metadata. This file contains the relevant source code, package dependencies, R version and other information.

#### 2. Push Bundle to Connect

The bundle is now pushed to Posit Connect server.

On successful deployment, `rsconnect` generates a folder in the original target directory called `rsconnect`. This file contains a `.dcf` file with information about the deployed application, such as the name, title, server address, account, URL and time.

If the application is re-deployed from the same directory, `rsconnect` checks for this file and can track multiple versions of the same application.

> [!TIP]
> A single bundle can be deployed to multiple servers or under different accounts. This can be specified under the *account* parameter in the `deployApp` function.

#### 3. Bundle Deployed on Posit Connect

Once the bundle is deployed, Posit Connect parses the manifest and restore all the package dependencies from the `packrat` lock file.