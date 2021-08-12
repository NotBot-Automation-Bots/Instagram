# Getting Started

This document explains how to successfully call Instagram Messaging with your app and get Instagram Business messages. It assumes you are familiar with the Graph API and Facebook Login, and know how to perform REST API calls.

### Before You Start

You will need access to the following:

-    An Instagram Business Account that fulfills the rollout criteria
-    A Facebook Page connected to that account
-    A Facebook Developer account that can perform Tasks with atleast "Moderate" level access on that Page
-    A registered Facebook App with Basic settings configured

Developers that are new to the Messenger Platform

-    Follow the step-by-step guide detailed below on how to generate Page Access Token, webhooks setup.
-    Learn about the various platform features and adopt those that suit your needs.
-    Ensure that you are implementing story mention correctly
-
Developers with prior experience on the Messenger Platform

-    Access token and webhooks concepts are similar. Instagram Messaging will require instagram_manage_messages in the Page Access Token and Instagram topic webhooks subscribed.
-    Most of the features are similar to Messenger API. Review the details on feature list and adopt those that suits your needs.
-    Ensure that you are implementing story mention correctly.

## 1. Configure Facebook Login
Add the Facebook Login product to your app in the App Dashboard.

You can leave all settings on their defaults. If you are implementing Facebook Login manually, enter your redirect_uri in the Valid OAuth redirect URIs field. If you will be using one of our SDKs, you can leave it blank.

Facebook login is required to get the appropriate access token on assets/pages that are not owned by the developer. For development purpose, you can also leverage Graph API Explorer to generate the appropriate access tokens for assets that you owned and skip to step 3.

## 2. Implement Facebook Login

Follow our Facebook Login documentation for your platform and implement Facebook Login into your app. Set up your implementation to request these permissions:

-    instagram_basic
-    instagram_manage_messages
-    pages_manage_metadata

## 3. Get a User Access Token

Once you've implemented Facebook Login, make sure you are signed into your Facebook Developer account, then access your app and trigger the Facebook Login modal. Remember, your Facebook Developer account must be able to perform Tasks with atleast "Moderate" level access on the Facebook Page connected to the Instagram account you want to query.

Once you have triggered the modal, click OK to grant your app the instagram_basic, instagram_manage_messages, and pages_manage_metadata permissions.

The API should return a User access token. Capture the token so your app can use it in the next few queries. If you are using the Graph API Explorer, it will be captured automatically and displayed in the Access Token field for reference:
![image](https://user-images.githubusercontent.com/40603380/129242532-edd894d9-3092-485b-8ed7-7340d618765e.png)

## 4. Get the User's Pages

Query the GET /me/accounts endpoint (this translates to GET /{user-id}/accounts, which perform a GET on the Facebook User node, based on your access token).
```sh
curl -i -X GET \
 "https://graph.facebook.com/v9.0/me/accounts?access_token={access-token}"
 ```
 This should return a collection of Facebook Pages that the current Facebook User can perform the MANAGE, CREATE_CONTENT, MODERATE, or ADVERTISE tasks on:
 ```sh
 {
  "data": [
    {
      "access_token": "EAAJjmJ...",
      "category": "App Page",
      "category_list": [
        {
          "id": "2301",
          "name": "App Page"
        }
      ],
      "name": "Metricsaurus",
      "id": "134895793791914",  // capture the Page ID
      "tasks": [
        "ANALYZE",
        "ADVERTISE",
        "MODERATE",
        "CREATE_CONTENT",
        "MANAGE"
      ]
    }
  ]
}
```
Capture the ID of the Facebook Page that's connected to the Instagram account that you want to query. Keep in mind that your app users may be able to perform tasks on multiple pages, so you eventually will have to introduce logic that can determine the correct Page ID to capture (or devise a UI where your app users can identify the correct Page for you).

## 5. Get the Page Access Token

In order to perform various IG Messaging API calls, you will need to use the associated Page Access Token (PAT) of the relevant IG account that has been previously granted via FB login flow.

Send a **GET** request to the _**/{page-id}**_ endpoint using your User access token. For example:
```sh
curl -i -X GET "https://graph.facebook.com/{page-id}?
  fields=access_token&
  access_token={user-access-token}"  
```
On success, your app gets this response:
```sh
{
  "access_token":"{page-access-token}",
  "id":"{page-id}"              
}  
```
-    If you used a short-lived User access token, the Page access token is valid for only 1 hour.
-    If you used a long-lived User access token, the Page access token has no expiration date.
To generate a long-lived Page acecss token, you can follow the guide [here](https://developers.facebook.com/docs/facebook-login/access-tokens/refreshing/#get-a-long-lived-page-access-token).

## 6. Enable Message Control Connected Tools Settings
In order to manage Instagram messages via API, IG business accounts will need to enable the connected tools toggle under message control settings.
![image](https://user-images.githubusercontent.com/40603380/129243164-7c637d19-e692-4e4d-9e9d-975e153e00c6.png)

## 7. Get the Instagram Business Account's Inbox Objects

Use the Page ID you captured and the Page Access Token (PAT) to query the GET /{page-id}/conversations?platform=instagram endpoint:
```sh
curl -i -X GET \
 "https://graph.facebook.com/v9.0/17841405822304914/conversations?platform=instagram&access_token={access-token}"
```
This should return the IDs of all the thread objects on the IG user:
```sh
{
  "data": [
    {
      "id": "aWdfZAG06MTpJR01lc3NhZA2VUaHJlYWQ6OTAwMTAxNDYyOTkyODI6MzQwMjgyMzY2ODQxNzEwMzAwOTQ5MTI4MTM2MDk5MDc1MzYyOTgx"
    },
    {
      "id": "aWdfZAG06MTpJR01lc3NhZA2VUaHJlYWQ6OTAwMTAxNDYyOTkyODI6MzQwMjgyMzY2ODQxNzEwMzAwOTQ5MTI4MTYzMzQ2MzE5NjM1NDcy"
    },
    {
      "id": "aWdfZAG06MTpJR01lc3NhZA2VUaHJlYWQ6OTAwMTAxNDYyOTkyODI6MzQwMjgyMzY2ODQxNzEwMzAwOTQ5MTI4MTk3MTY0NjI2NzAyMjMw"
    },
    {
      "id": "aWdfZAG06MTpJR01lc3NhZA2VUaHJlYWQ6OTAwMTAxNDYyOTkyODI6MzQwMjgyMzY2ODQxNzEwMzAwOTQ5MTI4MzkzNDI5MDYzMzkyNjU0"
    }
}
```
If you can perform this final query successfully, you should be able to perform queries using any of the Instagram Messaging API endpoints - just refer to our various guides and references to learn what each endpoint can do and what permission they require.

## Next Steps

-    Develop your app further so it can successfully use any other endpoints it needs, and keep track of the permissions each endpoint requires
-    Complete the webhook setup so it can receive real time notifications whenever user sends a message to the IG business account.
-    Complete the App Review process and request approval for all permissions your app will need so your app users can grant them while your app is in production.
