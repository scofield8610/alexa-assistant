# alexa-assistant

Implementation of the Google Assistant API for Alexa

# Release 3.0

### THIS SKILL IS FOR PERSONAL USE ONLY AND IS NOT ENDORSED BY GOOGLE OR AMAZON. WHILST THIS SKILL USES AN OFFICIAL GOOGLE API, IT WILL NEVER PASS AMAZON CERTIFICATION DUE TO THE WAY THE RESPONSES ARE HOSTED.


# What's New in this release

* Supports Node.js 14.x
* Supports for ASK CLI deployment all-in-one skill and CloudFormation stack with optional local building using docker
* Added support for device location converting Alexa skill device address to location coordinates using Google Maps Geocode API
* Increased supported locales list
* Replaced deprecated render template interface with simple cards display when text response available
* No longer need to upload `client_secret.json` as replaced by Lambda environmental variables
* Refactor code separating functionalities and upgrading to ASK SDK v2
* Added automated build release via GitHub actions

# Deployment Steps

0. Prerequisites

    * To deploy this skill, you will need the following tools:
        * [ASK CLI](https://developer.amazon.com/en-US/docs/alexa/smapi/quick-start-alexa-skills-kit-command-line-interface.html)
        * [Docker](https://docs.docker.com/get-docker/) (Optional to build Lambda dependencies locally)

    * To determine your Google OAuth2 client authorized redirect URIs, add your [Alexa Vendor ID](https://developer.amazon.com/settings/console/mycid) to the links below:
        * `https://layla.amazon.com/api/skill/link/<vendorId>`
        * `https://pitangui.amazon.com/api/skill/link/<vendorId>`
        * `https://alexa.amazon.co.jp/api/skill/link/<vendorId>`

    * Setup your Google environment by following the [Google-related](https://github.com/tartanguru/alexa-assistant-instructions/blob/master/fresh_install.md#enable-google-assistant-api-) installation instructions only, skipping all Amazon-related steps, and taking note of the [Project ID](https://console.cloud.google.com/cloud-resource-manager) that was created. Once completed, create your Google Maps API key as follow:
        * Enable the [Google Maps Geocoding API](https://console.cloud.google.com/apis/library/geocoding-backend.googleapis.com) under the same project.
        * Create a new [API key credential](https://console.cloud.google.com/apis/credentials) and edit it:
            * Change name to `google_assistant`
            * Leave application restrictions to none
            * Set API restrictions to the Geocoding API only.

    * Configure your `GoogleProjectId` and `GoogleMapsApiKey` in [`ask-resources.json`](ask-resources.json). To enable debug logs, add `LambdaDebug` set to `true` in the skill infrastructure cloudformation user config parameters.

    * Configure your Google Assistant API `clientId` and `clientSecret` in [`skill-package/accountLinking.json`](skill-package/accountLinking.json)

    * If your default language isn't English (US):
        * Configure your deploy `awsRegion` in [`ask-resources.json`](ask-resources.json) based on the table below:

            | Skill Language | Endpoint Region | Deploy Region |
            |:--------------:|:---------------:|:-------------:|
            | English (CA), English (US), French (CA), Portuguese (BR),<br>Spanish (MX), Spanish (US) | North America | `us-east-1` |
            | English (UK), French (FR), German, Italian, Spanish (ES) | Europe | `eu-west-1` |
            | English (IN) | India | `eu-west-1` |
            | English (AU), Japanese | Far East | `us-west-2` |

        * Add your specific skill locale:
            ```
            $ ask skill add-locales
            ? Please select at least one locale to add: en-GB

            The following skill locale(s) have been added according to your local project:
              Added locale en-GB.json from en-US's interactionModel
            Please check the added files above, and run "ask deploy" to deploy the changes.
            ```

    * If upgrading from v2:
        * Create ask hidden directory and states file adding the Skill ID listed under your [Alexa developer console](https://developer.amazon.com/alexa/console/ask). This will prevent duplicate skills from being created under your account.
            ```
            $ cat .ask/ask-states.json
            {
              "askcliStatesVersion": "2020-03-31",
              "profiles": {
                "default": {
                  "skillId": "<skillId>"
                }
              }
            }
            ```
        * Delete your existing `AlexaAssistant` CloudFormation stack under your [AWS console](https://console.aws.amazon.com/cloudformation/home).

1. Deploy the skill and the lambda function in one step:
    ```
    $ ask deploy
    Deploy configuration loaded from ask-resources.json
    Deploy project for profile [default]

    ==================== Deploy Skill Metadata ====================
    Skill package deployed successfully.
    Skill ID: <skillId>

    ==================== Build Skill Code ====================
    Skill code built successfully.
    Code for region default built to <skillPath>/.ask/lambda/build.zip successfully with build flow CustomBuildFlow.

    ==================== Deploy Skill Infrastructure ====================
    ✔ Deploy Alexa skill infrastructure for region "default"
    The api endpoints of skill.json have been updated from the skill infrastructure deploy results.
    Skill infrastructures deployed successfully through @ask-cli/cfn-deployer.

    ==================== Enable Skill ====================
    Skill is enabled successfully.
    ```

2. Update the skill account linking information using the Skill ID displayed in previous step:
    ```
    $ ask smapi update-account-linking-info -s <skillId> --account-linking-request file:skill-package/accountLinking.json

    Command executed successfully!
    ```

3. Enable the skill on your Alexa account:
    * Go to your [Alexa skill console](https://alexa.amazon.com/spa/index.html#skills/your-skills/?ref-suffix=ysa_gw)
    * Click on the "Google Assistant" skill under the "Dev Skills" tab
    * Click "Enable", activate the "Device Country and Postal Code" permission and go through the account linking process
