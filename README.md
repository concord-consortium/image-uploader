# Image Uploader

## Development

This repo is the result of a small spike to design and test integrating a generic image uploader component that would allow for the following:


### Uploader Component

- A component (in this repo) importable into `question-interactives` that would show a button that when clicked would show two choices under the button: "Upload from your computer" and "Upload from your mobile device".
- If the user chooses "Upload from your computer" the two choices are replaced by a drag-drop area with a label of "Drag an image file here to upload or click to select a file".  Once an image was given the Attachments API would be used to upload the file to S3.  The component would poll the read version image url and once found would use a callback given in the component props to pass back the url to the interactive component.
- If the user chooses "Upload from your mobile device" the Attachments API would be used to generate an upload url with a long lived signature (10 minutes?).  A QR code would be generated for an url in the form of `<external-device-uploader-page-url>?uploadUrl=<attachments-upload-url>` and would be displayed in place of the two choices.

### External Device Uploader Page

- A user would use their mobile device to scan the QR code generated in the uploader component which would load the external device uploader page (in this repo) which would show two choices: "Take a photo" and "Select an Image from Your Library".
- The "Take a photo" component would be lifted from the Vortex repo (https://github.com/concord-consortium/vortex/blob/master/src/mobile-app/components/camera.tsx)
- The "Select an Image..." would use a file input component with the mimetype set to `image/*` using the accept attribute.
- In either case once image data is available the page would use the passed `uploadUrl` query param to save the image data as a post request.
- The uploader component (as it would during its local upload) would be polling the read image url and updating its parent component when the image url resolved.


### DONE

- System architecture and this documentation
- Setup this repo
- Created an barebones uploader component and external device uploader page
- Updated the build configuration to generate a library for the uploader component and a S3 hostable page for the external device uploader page.
- Integrated the barebones uploader component into the `questions-interactive` repo to verify the library build worked

### TODO TO GO FROM SPIKE TO REAL VERSION (3.5 days effort)

- Setup Dev Environment
    - Locally use `npm link` to link the `dist/component` folder
    - Create a local branch in `question-interactives` and use `npm link @concord-consortium/image-uploader` to link to the local code
    - Add `import { ImageUploader } from "@concord-consortium/image-uploader";` to Labbook in `question-interactives` and then add `<ImageUploader />` before the existing `<UploadImage...>` component.
    - Build `questions-interactives` and test the stubbed uploader
- Stub/Wirefame Elements (1 day)
    - Create a stubbed uploader class that will be shared between the component and the uploader page.  Its constructor will take an url and it will expose a method to upload a passed in image (either by filename or buffer) and returns a Promise that resolves when the image is uploaded or upload fails
    - Add UI to interactive component that asks the user if they want to upload from the desktop or from another device.  Implement the desktop selector and uploader with a stubbed uploader that returns a static url to a kitten picture.  Test this in Lara and AP.
    - Create new uploader page as wireframe with no upload support and get it serving as static page on S3 using normal GitHub actions
    - Update new interactive component to generate QR code using the url to the uploader page with a fake attachments post url when the upload from other device option is selected. Test this in Lara and AP.  Use the existing code in Vortex for the QR Code generation (https://github.com/concord-consortium/vortex/blob/06b9424d96e66cedc3010c3e739bfc119a918cc0/src/lara-app/components/runtime.tsx#L55-L70)
- Implement Uploader (4 hours)
    - Add generation of Attachments read/write urls in component and update QR code generation
    - Implement file upload using `fetch` API (for easier Jest mocking)
    - Test this in Lara and AP
- Flesh out Component/Page UI (1 day)
    - Style wireframed component and page to match existing Lara/AP styling
    - Add watching of upload progress in interactive component and uploader page.
    - Add callback property to component to send url to parent component. On upload finished use the prop callback to set the url.
- Add Jest and Cypress Tests and setup test activities for QA (1 day)

### Run using HTTPS

Additional steps are required to run using HTTPS.

1. install [mkcert](https://github.com/FiloSottile/mkcert) : `brew install mkcert` (install using Scoop or Chocolatey on Windows)
2. Create and install the trusted CA in keychain if it doesn't already exist:   `mkcert -install`
3. Ensure you have a `.localhost-ssl` certificate directory in your home directory (create if needed, typically `C:\Users\UserName` on Windows) and cd into that directory
4. Make the cert files: `mkcert -cert-file localhost.pem -key-file localhost.key localhost 127.0.0.1 ::1`
5. Run `npm run start:secure` to run `webpack-dev-server` in development mode with hot module replacement

Alternately, you can run secure without certificates in Chrome:
1. Enter `chrome://flags/#allow-insecure-localhost` in Chrome URL bar
2. Change flag from disabled to enabled
3. Run `npm run start:secure:no-certs` to run `webpack-dev-server` in development mode with hot module replacement

### Building

If you want to build a local version run `npm build`, it will create the files in the `dist` folder.
You *do not* need to build to deploy the code, that is automatic.  See more info in the Deployment section below.

### Notes

1. Make sure if you are using Visual Studio Code that you use the workspace version of TypeScript.
   To ensure that you are open a TypeScript file in VSC and then click on the version number next to
   `TypeScript React` in the status bar and select 'Use Workspace Version' in the popup menu.

## Deployment

Production releases to S3 are based on the contents of the /dist folder and are built automatically by GitHub Actions
for each branch pushed to GitHub and each merge into production.

Merges into production are deployed to TDB

Other branches are deployed to TDB

To deploy a production release:

1. Increment version number in package.json
2. Create new entry in CHANGELOG.md
3. Run `git log --pretty=oneline --reverse <last release tag>...HEAD | grep '#' | grep -v Merge` and add contents (after edits if needed to CHANGELOG.md)
4. Run `npm run build`
5. Copy asset size markdown table from previous release and change sizes to match new sizes in `dist`
6. Create `release-<version>` branch and commit changes, push to GitHub, create PR and merge
7. Checkout master and pull
8. Checkout production
9. Run `git merge master --no-ff`
10. Push production to GitHub
11. Use https://github.com/concord-consortium/image-uploader/releases to create a new release tag

### Testing

Run `npm test` to run jest tests. Run `npm run test:full` to run jest and Cypress tests.

##### Cypress Run Options

Inside of your `package.json` file:
1. `--browser browser-name`: define browser for running tests
2. `--group group-name`: assign a group name for tests running
3. `--spec`: define the spec files to run
4. `--headed`: show cypress test runner GUI while running test (will exit by default when done)
5. `--no-exit`: keep cypress test runner GUI open when done running
6. `--record`: decide whether or not tests will have video recordings
7. `--key`: specify your secret record key
8. `--reporter`: specify a mocha reporter

##### Cypress Run Examples

1. `cypress run --browser chrome` will run cypress in a chrome browser
2. `cypress run --headed --no-exit` will open cypress test runner when tests begin to run, and it will remain open when tests are finished running.
3. `cypress run --spec 'cypress/integration/examples/smoke-test.js'` will point to a smoke-test file rather than running all of the test files for a project.

## License

Copyright 2018 (c) by the Concord Consortium and is distributed under the [MIT license](http://www.opensource.org/licenses/MIT).

See license.md for the complete license text.
