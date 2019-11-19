
Task 1 -- Deploy PetAppAuthDemoApp and configure Cognito to work with Okta as an external IdP
=================================================

### Step 1: Deploy PetAppAuthDemoApp

To perform this task, login to AWS account console, choose Cloud9 as your service and open pre-created Cloud9 environment. 
Your Cloud9 environment may not be seen under "You environments" but can be found under "Account environments".

The git repository https://github.com/aws-samples/amazon-cognito-example-for-external-idp already cloned to your Cloud9 environment. 

1.  Retrieve your AWS account ID:
```
aws sts get-caller-identity --query Account --output text
```
2.  Change directory to amazon-cognito-example-for-external-idp and copy env template to env.sh
```
cd amazon-cognito-example-for-external-idp
cp env.sh.template env.sh
ls
```
3.  Left click with your mouse on the env.sh and open it for editing:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image1.png" width="700" /> 

4.  Replace **$(aws configure get account)** with the account ID from the step 1 above.
5.  Replace **$(aws configure get region)** with **us-west-2** which is your current region. 
6.  Save the env.sh file:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image10.png" width="700" />  

7. Switch to the environments tab in your Cloud9 environment and run install.sh
```
./install.sh
```
8. When it completes, go to env.sh tab, comment out the "export APP_URL=http://localhost:3000" and uncomment "export APP_FRONTEND_DEPLOY_MODE=s3". This will deploy your applicaiton UI in newly created S3 bucket:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image11.png" width="700" /> 

9. Save env.sh file and run demploy.sh:
```
./deploy.sh
```
10. Accept proposed changes in the "Do you wish to deploy these changes (y/n)?" question by tuping "y" and enter.
11. Note the stack output. You will need it for your IdP configruation:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image12.png" width="700" /> 

You can open a new file tab in your Cloud9 environment and copy this info there.

Now we need to configure our IdP. 

### Step 2: Okta Directory and Application Setup

#### If you already have a developer account with Okta, please skip to Step 3

1. Sign up for a developer account on [Okta](https://developer.okta.com/) using your corporate credentials.
2. Activate your account and sign into your Okta domain *stated in the email*.
3. Go to the Admin dashboard by clicking on the **Admin** button on the top-right corner of the page.
4. In the Admin dashboard, go to the top-left of the page where it says **Developer Console** and change it to **Classic UI**.

![Console Page](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/dev-classicUI.png)

5. On the right-hand side of the page, under **Shortcuts**, click **Add Applications**.

![Add Applications](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/add-applications.png)

6. Select **Create New App** from the left side of the page.

![Create New Application](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/add-app2.png)

7. For Platform, select **Web** and enable **SAML 2.0** for the Sign in method. Then, press **Create**.

![Set SAML 2.0](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/app-integration.png)

8. Give the app a name and a logo (*optional*), then select **Next**.

![Set App Name and Logo](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/gen-settings.png)

9. The next page describes the SAML settings for your app.
10. The **Single sign on URL** will be your Cognito user pool App Integration domain with */saml2/idpresponse* appended. You can find it in the deploy.sh output value for "Single sign on URL / Assertion Consumer Service (ACS) URL".
   * Example: *https://yourDomainPrefix.auth.yourRegion.amazoncognito.com/saml2/idpresponse*
11. Make sure the **Use this for Recipient URL and Destination URL** box is checked.
12. For the **Audience URI (SP Entity ID)**, enter the urn for your Cognito user pool. This is the value of deploy.sh "Audience URI (SP Entity ID)".
13. Leave the **Default RelayState** blank.
14. Select *unspecified* for **Name ID format**.
15. Select *Okta username* for **Application username**.
16. Under **Attribute Statements**, configure the following:

    Name | Name format | Value
    :---: | :---: | :---:
    email | Unspecified | user.email
    firstName | Unspecified | user.firstName
    lastName | Unspecified | user.lastName

17. Under Group Attribute Statements, add the following:

    Name | Name format | Filter | value
    :---: | :---: | :---: | :---:
    groups | Unspecified | Starts with | pet-app


![SAML Settings](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/saml-settings.png)

18. Click **Next**.
19. Select *I'm an Okta customer adding an internal app* for **Are you a customer or partner?**.
20. Select *This is an internal app that we have created* for **App Type**.

![App Purpose](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/app-config.png)

21. Click **Finish**.
22. On the next screen download IdP metadata that you will use for Cognito integration with Okta:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image13.png" width="700" /> 

23. Go to the Assinments tab, click Assign then Assign to Groups:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image14.png" width="700" /> 

24. Click Assign for Everyone and click Done:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image15.png" width="700" /> 

Create two Okta groups: pet-app-admins and pet-app-users.

25. In the Okta portal go to the Directory -&gt; Groups and click Add Group:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image21.png" width="700" /> 

26. Create two groups, pet-app-admins and pet-app-users :

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image200.png" width="700" /> 

27. Go to the Directory -&gt; People and click Add People:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image21.png" width="700" /> 

28. Create two users, petappadmin1@example.com that belongs to pet-app-admins group and petappuser1@example.com that belongs to pet-app-users group:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image23.png" width="700" /> 

Now when we have Okta configured, we need to configure Cognito user pool SAML federation. 

### Step 3: Cognito SAML provider configuration

1. Login to you AWS account and go to Cognito service, choose “Manage User Pools” and choose your Cognito pool. 
2. Go to “Identity Providers” under “Federation”

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image16.png" width="700" /> 

3.  Click “SAML”, select AWS SSO metadata file, saved above, assign the name for your identity provider and click “Create Provider”:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image17.png" width="700" /> 

4. Go to “Attributes Mapping”, choose “SAML” tab and choose your okta identity provider you just created.
5. Add attributes mapping “groups” and choose “custom:groups” from the drop-down list as below.
SAML attribute “groups” will be included in the SAML assertion as result of the Attributes Mapping configuration in the step 10 above. The drop-down list contains your Cognito User Pool attributes. If your user pool has additional required attributes, you need to configure them to be included in the SAML assertion on AWS-SSO side as in step 8 above, then add them to the attribute mapping here. 
6. Click “Save Changes”.

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image24.png" width="700" /> 

7. Go to App client settings in the Cognito console, choose your application client and update Enabled Identity Providers to Okta. Click Save changes:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image26.png" width="700" /> 

Now let's test is all together. 

### Step 4: Test Sign in with SSO:

In your browser, open PetAppAuthDemo UI. The value of "appUrl" in the deploy.sh output is your application UI url. 

1. Open Chrome browser incognito window, right click to open developers tool, open a networking tab in developers tools and then navigate to the "appUrl" 
2. Signle Sign On”:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image25.png" width="700" /> 

3. Click on "okta" in "Sign in with your corporate ID" window.
4. Enter credentials for petappuser1@example.com. Answer all questions related to the first time Okta login.
5. User getting a SAML assertion from Okta, user will be redirected back to Cognito and will submit this SAML assertion to Cognito saml2/idpresponse endpoint:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image28.png" width="700" /> 

6. If you have any SAML viwer extention installed in your chrome browser, you can look into SAML asserion and see that user's group attributed included in the assertion:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image32.png" width="700" /> 

7. Cognito generates authorization code and client submits it to the application:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image35.png" width="700" /> 

8. After the application gets the authorization code from the URL, it issues a POST to the /token endpoint with the authorization code to exchange it for JWT tokens. Subsequently, client uses the JWT tokens to access the application’s API:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image35.png" width="700" /> 

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image33.png" width="700" /> 

9. You can look at the content of your JWT token by navigating to https://hwt.io , copy and paste your authorization header into Encoded form: 

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image34.png" width="700" /> 

As far as you can see, user's group attribute passed to the applicaion in JWT token. 

10. When you logged to the application, you can see that you're a member of pet-app-users, where you can create a new pet and see only the pets you own. 
11. Try to create a new pet:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image27.png" width="700" /> 

12. Log out from the application and sign in with petappadmin1@example.com user:

<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image37.png" width="700" /> 

13. Analize your JWT token and check what you can see when you logged in as administrator.

14. To explore further components have been built above, use AWS console to navigate to Cognito User pools, API Gateway deployments and Lambda functions.