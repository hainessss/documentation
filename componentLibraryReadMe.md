# Echo Component Library

Welcome to developing on the Echo Component Library platform.
Woo this is fun!

## Getting Started

To get started developing run `yarn`.

Then run `npm start`.

This will run through `src/components` and create the automated documentation.
Then it will serve up the documentation site. It will watch the src
and update as you make changes in the code. However, to update the
documentation you will have to re-run `npm start`.

## Directory Stucture

### docs

Contains the documentation static site.

### gulp

Write gulp tasks here.

### icons

Add icons to `icons/svg`

### src/components

Consumed components go here.

### src/containers

Higher order components go here.

### src/utils

Helper modules go here.


## Adding a components

To add a component simply run `npm run addComp`.  Give your component a name
when prompted. Use PascalCase in the following format. `EchoComponentName`
This will automatically wire it up for auto-documentation. Next time you run
`npm start` you will see your newly create component.

Each component will come with a spec file for tests, an index file for exporting the api
wrapper, a scss file for styles, a main component file, and an example directory for
creating a sample component. Add sub-components to a sub-directory named `/components`.

## Adding documentation.

Add general documentation to the project `README.md` and it will populate to the
static site.  Add proptype documentation by writing comments above the proptypes
in main components like so:

```
    EchoButton.propTypes = {
        /**
        * The text to be displayed in the button.
        */
        label: React.PropTypes.string.isRequired
    }
```

You can associate examples with components by writing a comment above them like so:

```
    /**
    * EchoComponent example
    *
    * @name SimpleEchoComponentExample
    * @component ../path/to/EchoComponent.js
    */
 ```
You can add comments about your component in the description section.  Specifying isWideView will render your examples above your sample code and allow space for wider components:

```
/**
 * Enter description.
 * isWideView
 * @name EchoSubHeader
 */
 ```

 Specifying isHidden in the component comments will hide the component from the docsite:

```
/**
 * Enter description.
 * isHidden
 * @name EchoSubHeader
 */
 ```

 Run `npm run docGen` if you ever want to update the documentation. It spits out
 the result in `/docs/docs.js` which is then consumed by the static site app in
 `/docs/index.js`.

## Z-INDEX LIST

2- EchoTooltip carets
5- EchoTreeOuter list items
10-EchoSubheader right column mobile view
21-EchoDrawer Shade
22-EchoDrawer Drawer
(?) 30-Echo Modal
15-EchoDropdown
(?) echoselect ,
(?) datepicker form
20-Echo Header
999-EchoTooltip form


## Testing

Write your tests in `src/components/EchoComponent/EchoComponent.spec.js`.

Run `npm test`.

Run `npm run test:watch` to re-run the test suite on code changes.

The tests use helper methods from AirBnb's `enzyme` library.
Documentation available at `https://github.com/airbnb/enzyme`;

## Deploying

To deploy the docs and publish the static site. Run `npm run deployDocs`.

Surge will ask you to login the first time you deploy. Credentials are:

xxxxx

The site is deployed at `http://echo-ux.surge.sh/`

To deploy the library run `npm run deploy`.

## Webpack

What the heck is it doing anyway.

### babel-loader

Compiling es6 to es5

### svg-sprite-loader ! svgo-loader

svg-sprite-loader creates sprite sheet for every imported svg and will attach it to </html>
at runtime. Consumers will be able to use icons either through `EchoIcon` or using the
`<use>` tag. svgo-loader optimizes the svgs, removing the un-needed junk.

### style-loader ! css-loader ! postcss-loader ! sass-loader

style-loader loads inline styles. css-loader creates css modules with
hashed class names. postcss-loader adds prefixing for browser compatability
and allows you to write scss selectors with dashes and use camelCase in the
javascript.  sass-loader compiles scss to css.

### markdown-loader

compiles markdown to html. (used in doc site).

## Common Bugs

If you see an error in the javascript console that says `Uncaught TypeError: Cannot read property 'name' of undefined` with a stack trace that runs through `PropsList`, the error is caused by having a prop listed in defaultProps that is not included as a PropType.

# Contributing

Contributions follow the workflow described [ here ](https://drive.google.com/a/redbookconnect.com/file/d/0Bz2KGejeubT9S0diRGNOa1Y2TjNXNzFjMXZMYWpaX1lCSWtR/view?usp=sharing)

To Contribute, create a ticket in the Echo Component(ECHO) project in JIRA. Include the name of the component in the summary [ComponentName] as well a short description of the ticket. Label the ticket with `echo`. Link the ticket to the epic `ui-component-version-1`. In the description of the ticket, please fill out and leave a link to one of the following short questionnaires.

For existing component modifications use [this template](https://docs.google.com/a/redbookconnect.com/document/d/1ayR9tww4XDmcxPMQmabfOx85rr8GELI40av5h_1sQSg/edit?usp=sharing).  For brand new component requests use [this template](https://docs.google.com/a/redbookconnect.com/document/d/1EdzIivLefpKOi4u05NtFPRtPWJj5LZ9NiM2MxpvGBRA/edit?usp=sharing). Add Alex Haines, Daniel Olasky, Alisa Illeeva, and Kevin Tomasso as watchers to the ticket.  Once the ticket is approved for development, fork the [repo](https://github.com/hotschedules/nxg-echo-component-lib).

All Echo tickets can be viewed at this [JIRA kanban board](https://jira.hotschedules.com/secure/RapidBoard.jspa?rapidView=353). If you need access to the JIRA project or Github repo, contact Mindy White, slack (@mindy.white), email (mindy.white@hotschedules.com). Create a feature branch named after your ticket number and move the ticket status to `in progress`. Once development is finished create a pull request on dev and move the ticket status to `in review`.  Tag Alex Haines as a reviewer.

Once the pull request is accepted Alex Haines will merge, publish the component to [http://echo-ux-dev.surge.sh/](http://echo-ux-dev.surge.sh/), and move the ticket to `waiting for deployment`.  At this point, design will review the component and deem acceptance or rejection.  Accepted components will be published to npm and [http://echo-ux.surge.sh/](http://echo-ux.surge.sh/). The ticket will then be moved to `done`.
