
Task 2 -- Create a read-only role
=================================================

In this task you will create a new group in Okta directory and will modify your app to have a read-ony role that will allow users that assigned to that Okta directly group to see pets owned by all users but not to be able to modify them.

### Step 1: Create new Okta group.

1.  Login to your Okta developer url which is found in the Okta Developer Activation Email from the previous task.

3.  Go to the Admin dashboard by clicking on the **Admin** button on the top-right corner of the page.

4.  In the Admin dashboard, go to the top-left of the page where it says **Developer Console** and change it to **Classic UI**.

    ![Console Page](https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/dev-classicUI.png)

5.  In the Okta portal go to the Directory -> Groups and click Add Group:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image21.png" width="700" />

6.  Create a new groups, pet-app-readonly :

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image1.png" width="700" />

7.  Go to the Directory -> People and click Add Person:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab1/media/image21.png" width="700" />

8.  Create a new user, petappreadonly1@example.com that belongs to pet-app-readonly group and set password by admin:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image2.png" width="700" />

    Now when we a new group and user Okta configured, we need to change our application. Please note that we don't have to change anything in Cognito or API Gateway.

### Step 2: Updating the application code

1.  Go to Cloud9 environment, find an open amazon-cognito-example-for-external-idp/lambda/api/src/app.ts file.

2.  Add readonlyGroupName to AppOptions as below:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image3.png" width="700" />

3.  Open amazon-cognito-example-for-external-idp/lambda/api/src/expressApp.ts.

4.  Assign readonlyGroupName value to the App:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image4.png" width="700" />

5.  Open amazon-cognito-example-for-external-idp/lambda/api/tests/app.test.ts.

6.  Add a constant for readonlyGroupName.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image10.png" width="700" />

7.  Add readonlyGroupName to the mocApp.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image11.png" width="700" />

8.  Go back to the app.ts and add readonlyGroupName to the authorizationMiddleware options:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image5.png" width="700" />

10. To allow read-only user to list all pets, edit /pets call as following:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image6.png" width="700" />

11. To allow read-only user to retrieve any pet, edit /pets/:petId call as following:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image7.png" width="700" />

    Next changes depend on the business logic we'd like to implement here. Below we'll prohibit any changes if user belongs to the read-only group, even it belongs to any other group.

12. Prohibit creating a new pet for read-only group:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image12.png" width="700" />

13. Prohibit put to /pets/:petId for read-only group:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image8.png" width="700" />

14. Prohibit deleting a pet:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/Cognito-Fine-Grained-Auth-External-IdP/lab2/media/image9.png" width="700" />

15. Go to File->Save All menu to save your changes.

16. Go to bash tab in your environment and redeploy your application.  Be sure you are in the following working directory **~/environment/amazon-cognito-example-for-external-idp**
```
./deploy.sh
```

17. After your re-deployed your applcation, close and open Chrome incognito window and test your application is working correctly by login with petappreadonly1@example.com and trying to list or view pets and to create or delete them.
