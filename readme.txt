# TRUENDO CMP Tag Template - README

## Important Note
For the most up-to-date and detailed instructions, please refer to the official TRUENDO documentation at [docs.truendo.com](https://docs.truendo.com). This README provides essential steps but the website contains comprehensive guidance.

Visit [www.truendo.com](https://www.truendo.com) for more information about TRUENDO and its services.

## Introduction
This README provides basic instructions for setting up the TRUENDO Consent Management Platform (CMP) using Google Tag Manager (GTM). TRUENDO supports two modes of operation: Basic Consent Mode and Advanced Consent Mode (this is the Google default). 

## Basic Consent Mode
In Basic Consent Mode, Google tags are prevented from loading until a user interacts with the consent banner. No data is transmitted to Google before the user consents.

### Steps to Implement Basic Consent Mode

1. **Enable Consent Mode in GTM:**
   - Navigate to the 'Admin' tab in GTM.
   - Select 'Container Settings'.
   - Tick the 'Enable consent overview (BETA)' checkbox under 'Additional Settings'.
   - Click 'Save'.

2. **Download TRUENDO Tag Template:**
   - Download the TRUENDO Tag Template from [here](https://github.com/truendo-tech/truendo-tag-template/blob/main/template.tpl).

3. **Import TRUENDO Tag Template:**
   - In GTM, navigate to the workspace tab of the container.
   - Click 'Templates' from the left panel.
   - Click the 'New' button.
   - In the 'Template Editor', click the Menu (three vertical dots) and select 'Import'.
   - Add the TRUENDO Tag Template and click 'Save'.

4. **Create a New Tag:**
   - Click 'Tags' in the left panel.
   - Click the 'New' button.
   - Rename the tag to 'TRUENDO' (optional for better tracking).

5. **Configure the TRUENDO Tag:**
   - Click the Edit (pencil) button in the 'Tag Configuration' pane.
   - Select 'Custom' and then the TRUENDO tag.
   - Tick the 'Inject TRUENDO via GTM' and 'Enable TRUENDO triggers' checkboxes.
   - Add your Site ID to the TRUENDO tag (obtainable from the TRUENDO Console).

6. **Create Custom Triggers:**
   - Click 'Triggers' in the left panel.
   - Create new triggers with the following details:
     - `truendo_initialized` for event `truendo_initialized`
     - `truendo_marketing` for event `truendo_cc_marketing`
     - `truendo_preferences` for event `truendo_cc_preferences`
     - `truendo_social_content` for event `truendo_cc_social_content`
     - `truendo_social_sharing` for event `truendo_cc_social_sharing`
     - `truendo_statistics` for event `truendo_cc_statistics`

7. **Configure Tags to Work in Basic Consent Mode:**
   - For each tag, expand the 'Consent Settings' and select 'Require additional consent for tag to fire'.
   - Add the required consent permissions and corresponding TRUENDO triggers.

8. **Save and Publish:**
   - Click 'Save' in the Tag Configuration.
   - Publish the changes in GTM.

## Advanced Consent Mode (Google Default)
In Advanced Consent Mode, Google tags load with the consent mode API when a user opens the website or app. Full measurement data is only sent upon user consent.

### Steps to Implement Advanced Consent Mode

1. **Enable Consent Mode in GTM:**
   - Navigate to the 'Admin' tab in GTM.
   - Select 'Container Settings'.
   - Tick the 'Enable consent overview (BETA)' checkbox under 'Additional Settings'.
   - Click 'Save'.

2. **Download TRUENDO Tag Template:**
   - Download the TRUENDO Tag Template from [here](https://github.com/truendo-tech/truendo-tag-template/blob/main/template.tpl).

3. **Import TRUENDO Tag Template:**
   - In GTM, navigate to the workspace tab of the container.
   - Click 'Templates' from the left panel.
   - Click the 'New' button.
   - In the 'Template Editor', click the Menu (three vertical dots) and select 'Import'.
   - Add the TRUENDO Tag Template and click 'Save'.

4. **Create a New Tag:**
   - Click 'Tags' in the left panel.
   - Click the 'New' button.
   - Rename the tag to 'TRUENDO' (optional for better tracking).

5. **Configure the TRUENDO Tag:**
   - Click the Edit (pencil) button in the 'Tag Configuration' pane.
   - Select 'Custom' and then the TRUENDO tag.
   - Tick the 'Inject TRUENDO via GTM' checkbox.
   - Add your Site ID to the TRUENDO tag (obtainable from the TRUENDO Console).

6. **Set Trigger:**
   - Click the Edit button in the 'Triggering' pane.
   - Select 'Consent Initialization - All Pages'.
   - Click 'Save'.

7. **Save and Publish:**
   - Click 'Save' in the Tag Configuration.
   - Publish the changes in GTM.

## Support
For further assistance, refer to the TRUENDO Documentation or contact TRUENDO support.

## Links
- [TRUENDO Documentation](https://docs.truendo.com)
- [TRUENDO Console](https://console.truendo.com)
- [TRUENDO Official Website](https://www.truendo.com)
- [Download TRUENDO Tag Template](https://github.com/truendo-tech/truendo-tag-template/blob/main/template.tpl)

Last Updated: June 7, 2024

