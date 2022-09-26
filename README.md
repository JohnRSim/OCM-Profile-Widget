
![image](https://user-images.githubusercontent.com/364208/192087711-527bd390-7089-437e-91c4-ca4436cbcf2a.png)

# OCM-Profile-Widget (In Development)

A quick profile widget that adds support to show and enable users to update their profile image and their social links with all the data coming from the OCM User API Services.


#### Enables users the ability to:

- View user profile imge and social links
- Enable user to manage their social links
- Enable user to change their avatar

As you can have unlimited user license on OCM - if a user is defined as a OCM user in IDCS - they will appear within this app providing a basic sample of social capabilities that OCM can provide OOTB with no custom middleware.

#### Filesize
gzip: 16.32 KiB 


# Check the blog for the latest updates

https://bitmapbytes.com/hidden-presence-service-with-oracle-content-management/

# Guide
Authentication currently uses OAuth2 against IDCS you will need to setup an IDCS application first - Please use this guide
https://docs.oracle.com/en/cloud/paas/content-cloud/solutions/integrate-oracle-content-management-using-oauth.html#GUID-49F31EE7-D7C0-4EE4-A9E8-B610A7E816B1

# CORS
If you are runnng localhost you will need to disable CORS I use the following flag with chrome and create a custom user dir "ChromeFiles" without this --disable-web-security will not work.
```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --allow-running-insecure-content --disable-web-security --user-data-dir=C:\ChromeFiles
```
If you have this running on a domain you can update your CORS settings on OCM 

_System >> Security_

And add the front and backchannel with your domain.

![image](https://user-images.githubusercontent.com/364208/190382422-750d662a-03c1-49de-b765-a31260e14326.png)

# Configuration
Custom Configurations for the build you will need to update.

### /src/lib/config/oce.json
```
{
    "contentServer": "https://<OCMInstance>.cec.ocp.oraclecloud.com"
}
```

### /src/lib/config/idcs.json
```
{
    "idcsUrl": "https://<IDCSInstance>.identity.oraclecloud.com",
    "clientId": "<IDCS_APP_CLIENTID>",
    "clientSecret": "<IDCS_APP_SECRETKEY>",
    "oauthScopeUrl": "<IDCS_APP_SCOPE>"
}
```

# Test Locally 
Once you have the above configured you can run the following command and test to confirm the widget works.
```
npm run dev
```

# Build
Once tested run the build script this will build the webcomponents into the ./dist folder
```
npm run build
```
# Deploy Headless
In the dist folder insert script into the html head tag or import and then reference the custom element.

Props info coming soon..

# Deploy OCM Sites

- In the component folder download the zipped package.
- in ./dist/ rename ocm-profile-widget.umd.cjs to ocm-profile-widget.umd.js
- In the Zip file replace with your build .js files
```
BB-Profile-Widget.zip\BB-Profile-Widget\assets\build\ocm-profile-widget.js
BB-Profile-Widget.zip\BB-Profile-Widget\assets\build\ocm-profile-widget.umd.js
```

- Upload to OCM Components and assign members.
- In sites drag and drop the component into a page.

# Notes
- Template (Complete)
- Inital Api Services Call Setup (Complete)
- App UI & Headless integration 
- OCM Sites Component Sample


## Enahancement Requests!! 
Yup... you want - just raise an issue and mark it as an enhancement and I'll add it to the backlog :) 
