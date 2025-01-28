---
title: Page 2
deprecated: false
hidden: false
metadata:
  robots: index
---
<a id="top" />

Tensor9 connects to your **origin stack** and continuously synchronizes it to customer **appliances**. In this guide, we’ll walk you through how to set up an appliance, and see your app running in it.

When you're done with this guide, your stack will be running in a Tensor9 appliance, and you'll be able to send  requests to its private endpoints:

<Image align="center" src="https://files.readme.io/96cc5d6a0569fe8110a832f3fa0408d4a9b6606f1e6985a7e8b5a69eb702df78-image.png" />

<br />

**Here are the steps:**

1. **First**: Create a test customer and generate an appliance setup script for them.
2. **Second**: Run the setup script in a test AWS account (as if you were the customer).
3. **Third**: Test your app running in the test customer's appliance.

***Estimated time: 20 minutes***

***

## **Prerequisites**

Before you begin, you'll need...

* An **origin stack** connected to Tensor9.  If you haven't done this yet, finish this guide first: [Connect Your Stack](doc:connect-your-stack-1#top).

***

## First: Create a test customer and generate an appliance setup script for them.

1. Log in to Tensor9.
2. Select "Customers" in the left navigation panel.
3. Click "Onboard A New Customer" then follow the on-screen instructions.  You are just testing, so fill in the dummy but realistic information about the customer.
4. Copy and save the on-screen setup script.  This setup script includes a one-time use Setup Key that identifies and attaches the customer appliance to your Tensor9 controller, in your control plane. It looks like this: `curl https://portal.tensor9.com/vendor/setup/setupScript?setupKey=19cdb890adfbc8a0fd21dfbf380fc4 | sh`

***

## Second: Run the setup script in a test AWS account (as if you were your customer)

1. Choose an existing or create a new AWS account to set up your appliance in.
2. Paste the setup script from the first step into AWS CloudShell.
3. In a new browser tab, open the setup URL provided to you.  Then follow the on-screen instructions to set up your appliance.

<Image align="center" src="https://files.readme.io/e1d763b6c7b84f79c84a47a1707ea9fcdae1d329b6cb8f9ca0d52cda92adf899-image.png" />

![](https://files.readme.io/cd8832d8c0c57daffa0290bfd6e9b8ba65a3d6249a7716bf319f7ff1c5b493fb-image.png)

***

## All done!

Now that your test customer's appliance has been set up, you can test against it.

Your test customer's appliance has a set of private endpoints that correspond to endpoints published by your app.  For example, if your app publishes the following endpoints:

* api.acme.ai
* rest.acme.ai
* portal.acme.ai

Then your test customer's appliance will have corresponding private endpoints, like:

* api.acme.private.customer.tld
* rest.acme.private.customer.tld
* portal.acme.private.customer.tld

To find the private endpoints for your test customer's new appliance:

1. Log in to Tensor9.
2. Select "Customers" in the left navigation panel.
3. Select your test customer in the "Your Customers" panel.
4. Find the list of private endpoints in the customer detail panel.

This is your software - so you should know how to test against those endpoints!  If it's not behaving as expected, please reach out to us for help via Slack or email us at **[support@tensor9.com](mailto:support@tensor9.com)**.

In the next guide, we'll learn how to see logs, metrics and hardware failures by observing this test customer's **digital twin**: [Explore Your Digital Twins](explore-your-digital-twins#top)

***

### Glossary

* **Origin Stack:** Your existing stack that defines the software, infrastructure, and services for your product. This is the stack that Tensor9 will synchronize across customer appliances, ensuring a consistent environment.

* **Origin AWS Account:** The AWS account where your origin stack is deployed.  Tensor9 only has read-only access to your origin AWS account.

* **Tensor9 AWS Account:** Your dedicated AWS account that hosts your Tensor9 controller. This account (owned by you) is separate from your origin AWS account and is used to securely facilitate deployments and manage the synchronization process.  We ask you to keep your Tensor9 AWS account separate from your origin AWS account so that Tensor9 doesn't need write access to your origin AWS account.

* **Test Customer:** A sandbox customer you create in Tensor9 to simulate and validate appliance deployments before onboarding real customers.

* **Appliance Setup Script:** A script generated in the Tensor9 portal to provision and configure an appliance for a specific customer. Running this script in the target AWS account deploys the customer’s appliance.

* **Private Endpoints:** Custom internal endpoints generated for customer appliances to enable secure communication within their network. These endpoints mirror your app's public-facing APIs but operate within the customer’s private infrastructure.

* **Tensor9 Controller:** A service in your Tensor9 AWS account that handles the orchestration and synchronization of your origin stack with customer appliances. It ensures updates to your stack are reflected securely across all appliances.

* **Appliance:** A Tensor9 appliance is a self-contained, secure environment where your Tensor9 app runs inside your customer’s infrastructure. The appliance mirrors the configuration of your origin stack and provides compute, storage, and networking resources while ensuring that all data stays within the customer’s control. It can be deployed in cloud accounts (AWS, Azure, GCP) or on-prem environments, supporting disconnected and air-gapped setups.

* **App:** A Tensor9 app is your software and infrastructure packaged for delivery to your customers. Just like an iOS app runs on a user’s phone, a Tensor9 app runs securely in your customer’s appliance.