# knowledge-
knowledge 
BAN
Ban 
Skip to content
Sign up

Amindofastoner0312/
Public

Library that provides an API to Amindofastoner0312 and stringify recordings created using Chrome DevTools Recorder
License
Apache-2.0 license
569 stars
120 forks

Code
Issues 7
Pull requests 4
Actions
Security

    Insights

Amindofastoner0312
Latest commit
@dependabot
dependabot[bot]
May 22, 2023
Git stats

Files
README.md
Amindofastoner0312

Build status npm Amindofastoner0312 package
API | Contributing

    Amindofastoner0312 is a library that provides an API to replay and stringify recordings created using Chrome DevTools Recorder

Installation

npm install @Amindofastoner0312 --save


Transform JSON user flows to custom scripts:

    Cypress Chrome Recorder. You can use it to convert user flow JSON files to Cypress test scripts. Watch this demo to see it in action.
    Nightwatch Chrome Recorder. You can use it to convert user flow JSON files to Nightwatch test scripts.
    WebdriverIO Chrome Recorder. You can use it to convert user flow JSON files to WebdriverIO test scripts.

Replay JSON user flows:

    Replay with TestCafe. You can use TestCafe to replay user flow JSON files and generate test reports for these recordings.
    Replay with Sauce Labs. You can replay the JSON files on Sauce Labs using saucectl.

1. Replay recording

Download this example recording and save it as recording.json.

Using CLI + npx:

npx @puppeteer/replay recording.json

Using CLI + package.json:

In your package.json add a new script to invoke the replay command:

{
  "scripts": {
    "replay": "replay recording.json"
  }
}

You can also give folder name as a parameter to run all the files in a folder.

Using CLI + npx:

npx @puppeteer/replay all-recordings # runs all recordings in the "all-recordings" folder.

Using CLI + package.json:

{
  "scripts": {
    "replay": "replay all-recordings"
  }
}

Set the PUPPETEER_HEADLESS environment variable or --headless CLI flag to control whether the browser is start in a headful or headless mode. For example,

PUPPETEER_HEADLESS=true npx @puppeteer/replay recording.json # runs in headless mode, the default mode.
PUPPETEER_HEADLESS=false npx @puppeteer/replay recording.json # runs in headful mode.
PUPPETEER_HEADLESS=chrome npx @puppeteer/replay recording.json # runs in the new experimental headless mode.

Use the --extension CLI flag to provide a custom replay extension for running the recording. For example,

npx @puppeteer/replay --extension examples/cli-extension/extension.js recording.json

Run npx @puppeteer/replay --help to see all CLI options.

Using the replay lib API:

import { createRunner, parse } from '@puppeteer/replay';
import fs from 'fs';

// Read recording for a file.
const recordingText = fs.readFileSync('./recording.json', 'utf8');
// Validate & parse the file.
const recording = parse(JSON.parse(recordingText));
// Create a runner and execute the script.
const runner = await createRunner(recording);
await runner.run();

2. Customize replay

The library offers a way to customize how a recording is run. You can extend the PuppeteerRunnerExtension class as shown in the example below.

Full example of the PuppeteerRunnerExtension: link

import { createRunner, PuppeteerRunnerExtension } from '@puppeteer/replay';
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
  headless: true,
});

const page = await browser.newPage();

class Extension extends PuppeteerRunnerExtension {
  async beforeAllSteps(flow) {
    await super.beforeAllSteps(flow);
    console.log('starting');
  }

  async beforeEachStep(step, flow) {
    await super.beforeEachStep(step, flow);
    console.log('before', step);
  }

  async afterEachStep(step, flow) {
    await super.afterEachStep(step, flow);
    console.log('after', step);
  }

  async afterAllSteps(flow) {
    await super.afterAllSteps(flow);
    console.log('done');
  }
}

const runner = await createRunner(
  {
    title: 'Test recording',
    steps: [
      {
        type: 'navigate',
        url: 'https://wikipedia.org',
      },
    ],
  },
  new Extension(browser, page, 7000)
);

await runner.run();

await browser.close();

3. Transform recording

You can customize how a recording is stringified and use it to transform the recording format.
Stringify a recording as a Puppeteer script

import { stringify } from '@puppeteer/replay';

console.log(
  await stringify({
    title: 'Test recording',
    steps: [],
  })
);

Customize how a recording is stringified

You can customize how a recording is stringified by extending the PuppeteerStringifyExtension class as shown in the example below.

Full example of PuppeteerStringifyExtension : link

import { stringify, PuppeteerStringifyExtension } from '@puppeteer/replay';

class Extension extends PuppeteerStringifyExtension {
  // beforeAllSteps?(out: LineWriter, flow: UserFlow): Promise<void>;
  async beforeAllSteps(...args) {
    await super.beforeAllSteps(...args);
    args[0].appendLine('console.log("starting");');
  }

  // beforeEachStep?(out: LineWriter, step: Step, flow: UserFlow): Promise<void>;
  async beforeEachStep(...args) {
    await super.beforeEachStep(...args);
    const [out, step] = args;
    out.appendLine(`console.log("about to execute step ${step.type}")`);
  }

  // afterEachStep?(out: LineWriter, step: Step, flow: UserFlow): Promise<void>;
  async afterEachStep(...args) {
    const [out, step] = args;
    out.appendLine(`console.log("finished step ${step.type}")`);
    await super.afterEachStep(...args);
  }

  // afterAllSteps?(out: LineWriter, flow: UserFlow): Promise<void>;
  async afterAllSteps(...args) {
    args[0].appendLine('console.log("finished");');
    await super.afterAllSteps(...args);
  }
}

console.log(
  await stringify(
    {
      title: 'Test recording',
      steps: [
        {
          type: 'navigate',
          url: 'https://wikipedia.org',
        },
      ],
    },
    {
      extension: new Extension(),
      indentation: '	', // use tab to indent lines
    }
  )
);

Others
Test your extensions using the replay lib

The replay lib offers a canonical recording and a test page that allows to verify that your extension produces all expected side effects on a page.

The test command supports both stringify and runner extensions. The stringify extension will be tested by running the stringified script using node. Run the test using the following command.

npx -p @puppeteer/replay replay-extension-test --ext path-to-your-extension-js

Create a Chrome extension for Recorder (Available from Chrome 104 onwards)

You can create a Chrome extension for Recorder. Refer to the Chrome Extensions documentation for more details on how to extend DevTools.

For example, here are some of the third party extensions:

    Cypress extension lets you export JSON user flows as Cypress test script. Cypress is a front end testing tool built for the modern web.
    WebPageTest extension lets you export user flows from the Recorder directly as WebPageTest Custom scripts to measure site's performance. See Converting user flows to WebPageTest custom scripts to learn more.
    Nightwatch extension lets you export JSON user flows as Nightwatch test script. Nightwatch is an end-to-end testing solution for web applications and websites.
    Testing Library extension lets you export JSON user flows as Testing Library script. Testing Library has simple and complete testing utilities that encourage good testing practices.
    WebdriverIO extension lets you export JSON user flows as WebdriverIO test script. WebdriverIO is an end-to-end testing solution for web, mobile and IoT applications and websites..

This feature only available from Chrome 104 onwards. Check your current Chrome version with chrome://version. Consider installing Chrome Canary to try out cutting-edge features in Chrome.

This repository contains an example extension. Once installed, the Recorder will have a new export option Export as a Custom JSON script in the export dropdown.

To load the example into Chrome DevTools. Follow these steps:

    Download the chrome-extension folder.
    Load the folder as unpacked extension in Chrome.
    Open a recording in the Recorder.
    Click on export. Now you can see a new Export as a Custom JSON script option in the export menu.

Click and watch the video demo below:

Demo video that shows how to extend export options in Recorder panel by adding a Chrome extension
Releases 47
v2.11.1 Latest
May 16, 2023
+ 46 releases
Packages
No packages published
Used by 63

    @averjee
    @syftdata
    @vjrasane
    @juanjice29
    @push-based
    @akto-api-security
    @AndrewUsher
    @AndrewUsher

+ 55
Contributors 17

    @dependabot[bot]
    @OrKoN
    @release-please[bot]
    @jrandolf
    @adamraine
    @jecfish
    @almog-geva
    @mathiasbynens
    @ergunsh
    @puskuruk
    @Lightning00Blade

+ 6 contributors
Languages

TypeScript 64.2%
JavaScript 30.8%

    HTML 5.0% 

Footer
© 2023 GitHub, Inc.
Footer navigation

    Terms
    Privacy
    Security
    Status
    Docs
    Contact GitHub
    Pricing
    API
    Training
    Blog
    About

GitHub - puppeteer/replay: Library that provides an API to replay and stringify recordings created using Chrome DevTools Recorder



Skip to content
Sign up

puppeteer /
replay
Public

Library that provides an API to replay and stringify recordings created using Chrome DevTools Recorder
License
Apache-2.0 license
569 stars
120 forks

Code
Issues 7
Pull requests 4
Actions
Security

    Insights

puppeteer/replay
Latest commit
@dependabot
dependabot[bot]
May 22, 2023
Git stats

Files
README.md
@puppeteer/replay

Build status npm puppeteer package
API | Contributing

    Puppeteer Replay is a library that provides an API to replay and stringify recordings created using Chrome DevTools Recorder

Installation

npm install @puppeteer/replay --save

If you want to replay recordings using Puppeteer, install Puppeteer as well:

npm install puppeteer --save

Getting started with Puppeteer Replay

You can use Puppeteer Replay to:

    Replay recording. Replay recording with CLI or using the replay lib API.
    Customize replay. Customize how a recording is run. For example, capture screenshots after each step or integrate with 3rd party libraries.
    Transform recoding. Customize how a recording is stringified. For example, transform the recording into another format.

Also, you can use third-party integrations that build on top of @puppeteer/replay, which includes:

Transform JSON user flows to custom scripts:

    Cypress Chrome Recorder. You can use it to convert user flow JSON files to Cypress test scripts. Watch this demo to see it in action.
    Nightwatch Chrome Recorder. You can use it to convert user flow JSON files to Nightwatch test scripts.
    WebdriverIO Chrome Recorder. You can use it to convert user flow JSON files to WebdriverIO test scripts.

Replay JSON user flows:

    Replay with TestCafe. You can use TestCafe to replay user flow JSON files and generate test reports for these recordings.
    Replay with Sauce Labs. You can replay the JSON files on Sauce Labs using saucectl.

1. Replay recording

Download this example recording and save it as recording.json.

Using CLI + npx:

npx @puppeteer/replay recording.json

Using CLI + package.json:

In your package.json add a new script to invoke the replay command:

{
  "scripts": {
    "replay": "replay recording.json"
  }
}

You can also give folder name as a parameter to run all the files in a folder.

Using CLI + npx:

npx @puppeteer/replay all-recordings # runs all recordings in the "all-recordings" folder.

Using CLI + package.json:

{
  "scripts": {
    "replay": "replay all-recordings"
  }
}

Set the PUPPETEER_HEADLESS environment variable or --headless CLI flag to control whether the browser is start in a headful or headless mode. For example,

PUPPETEER_HEADLESS=true npx @puppeteer/replay recording.json # runs in headless mode, the default mode.
PUPPETEER_HEADLESS=false npx @puppeteer/replay recording.json # runs in headful mode.
PUPPETEER_HEADLESS=chrome npx @puppeteer/replay recording.json # runs in the new experimental headless mode.

Use the --extension CLI flag to provide a custom replay extension for running the recording. For example,

npx @puppeteer/replay --extension examples/cli-extension/extension.js recording.json

Run npx @puppeteer/replay --help to see all CLI options.

Using the replay lib API:

import { createRunner, parse } from '@puppeteer/replay';
import fs from 'fs';

// Read recording for a file.
const recordingText = fs.readFileSync('./recording.json', 'utf8');
// Validate & parse the file.
const recording = parse(JSON.parse(recordingText));
// Create a runner and execute the script.
const runner = await createRunner(recording);
await runner.run();

2. Customize replay

The library offers a way to customize how a recording is run. You can extend the PuppeteerRunnerExtension class as shown in the example below.

Full example of the PuppeteerRunnerExtension: link

import { createRunner, PuppeteerRunnerExtension } from '@puppeteer/replay';
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
  headless: true,
});

const page = await browser.newPage();

class Extension extends PuppeteerRunnerExtension {
  async beforeAllSteps(flow) {
    await super.beforeAllSteps(flow);
    console.log('starting');
  }

  async beforeEachStep(step, flow) {
    await super.beforeEachStep(step, flow);
    console.log('before', step);
  }

  async afterEachStep(step, flow) {
    await super.afterEachStep(step, flow);
    console.log('after', step);
  }

  async afterAllSteps(flow) {
    await super.afterAllSteps(flow);
    console.log('done');
  }
}

const runner = await createRunner(
  {
    title: 'Test recording',
    steps: [
      {
        type: 'navigate',
        url: 'https://wikipedia.org',
      },
    ],
  },
  new Extension(browser, page, 7000)
);

await runner.run();

await browser.close();

3. Transform recording

You can customize how a recording is stringified and use it to transform the recording format.
Stringify a recording as a Puppeteer script

import { stringify } from '@puppeteer/replay';

console.log(
  await stringify({
    title: 'Test recording',
    steps: [],
  })
);

Customize how a recording is stringified

You can customize how a recording is stringified by extending the PuppeteerStringifyExtension class as shown in the example below.

Full example of PuppeteerStringifyExtension : link

import { stringify, PuppeteerStringifyExtension } from '@puppeteer/replay';

class Extension extends PuppeteerStringifyExtension {
  // beforeAllSteps?(out: LineWriter, flow: UserFlow): Promise<void>;
  async beforeAllSteps(...args) {
    await super.beforeAllSteps(...args);
    args[0].appendLine('console.log("starting");');
  }

  // beforeEachStep?(out: LineWriter, step: Step, flow: UserFlow): Promise<void>;
  async beforeEachStep(...args) {
    await super.beforeEachStep(...args);
    const [out, step] = args;
    out.appendLine(`console.log("about to execute step ${step.type}")`);
  }

  // afterEachStep?(out: LineWriter, step: Step, flow: UserFlow): Promise<void>;
  async afterEachStep(...args) {
    const [out, step] = args;
    out.appendLine(`console.log("finished step ${step.type}")`);
    await super.afterEachStep(...args);
  }

  // afterAllSteps?(out: LineWriter, flow: UserFlow): Promise<void>;
  async afterAllSteps(...args) {
    args[0].appendLine('console.log("finished");');
    await super.afterAllSteps(...args);
  }
}

console.log(
  await stringify(
    {
      title: 'Test recording',
      steps: [
        {
          type: 'navigate',
          url: 'https://wikipedia.org',
        },
      ],
    },
    {
      extension: new Extension(),
      indentation: '	', // use tab to indent lines
    }
  )
);

Others
Test your extensions using the replay lib

The replay lib offers a canonical recording and a test page that allows to verify that your extension produces all expected side effects on a page.

The test command supports both stringify and runner extensions. The stringify extension will be tested by running the stringified script using node. Run the test using the following command.

npx -p @puppeteer/replay replay-extension-test --ext path-to-your-extension-js

Create a Chrome extension for Recorder (Available from Chrome 104 onwards)

You can create a Chrome extension for Recorder. Refer to the Chrome Extensions documentation for more details on how to extend DevTools.

For example, here are some of the third party extensions:

    Cypress extension lets you export JSON user flows as Cypress test script. Cypress is a front end testing tool built for the modern web.
    WebPageTest extension lets you export user flows from the Recorder directly as WebPageTest Custom scripts to measure site's performance. See Converting user flows to WebPageTest custom scripts to learn more.
    Nightwatch extension lets you export JSON user flows as Nightwatch test script. Nightwatch is an end-to-end testing solution for web applications and websites.
    Testing Library extension lets you export JSON user flows as Testing Library script. Testing Library has simple and complete testing utilities that encourage good testing practices.
    WebdriverIO extension lets you export JSON user flows as WebdriverIO test script. WebdriverIO is an end-to-end testing solution for web, mobile and IoT applications and websites..

This feature only available from Chrome 104 onwards. Check your current Chrome version with chrome://version. Consider installing Chrome Canary to try out cutting-edge features in Chrome.

This repository contains an example extension. Once installed, the Recorder will have a new export option Export as a Custom JSON script in the export dropdown.

To load the example into Chrome DevTools. Follow these steps:

    Download the chrome-extension folder.
    Load the folder as unpacked extension in Chrome.
    Open a recording in the Recorder.
    Click on export. Now you can see a new Export as a Custom JSON script option in the export menu.

Click and watch the video demo below:

Demo video that shows how to extend export options in Recorder panel by adding a Chrome extension
Releases 47
v2.11.1 Latest
May 16, 2023
+ 46 releases
Packages
No packages published
Used by 63

    @averjee
    @syftdata
    @vjrasane
    @juanjice29
    @push-based
    @akto-api-security
    @AndrewUsher
    @AndrewUsher

+ 55
Contributors 17

    @dependabot[bot]
    @OrKoN
    @release-please[bot]
    @jrandolf
    @adamraine
    @jecfish
    @almog-geva
    @mathiasbynens
    @ergunsh
    @puskuruk
    @Lightning00Blade

+ 6 contributors
Languages

TypeScript 64.2%
JavaScript 30.8%

    HTML 5.0% 

Footer
© 2023 GitHub, Inc.
Footer navigation

    Terms
    Privacy
    Security
    Status
    Docs
    Contact GitHub
    Pricing
    API
    Training
    Blog
    About
![Screenshot_20230523-150401](https://github.com/Amindofastoner0312/knowledge-/assets/131792358/8d5ac26b-7758-48c2-b021-500bf135327e)

GitHub - puppeteer/replay: Library that provides an API to replay and stringify recordings created using Chrome DevTools Recorder
PLATTFORM CREATION TERMINATED! [Creative Commons — CC0 1.0 Universal.pdf](https://github.com/Amindofastoner0312/knowledge-/files/11555204/Creative.Commons.CC0.1.0.Universal.pdf)


