## Authentication in custom force.com site

Manage Authentication in custom force.com site; use account/contact/custom object as your site user storage.
This managed package use session to save user identity and based on <a href="https://github.com/sf-cn/Salesforce-Site-Session" target="_blank">Custom Session Management Package</a>

## Getting started

### Install Managed Package
<a href="https://login.salesforce.com/packaging/installPackage.apexp?p0=04t7F000000ZeWc" target="_blank">
  Deploy to Production
</a>
<br /><br />
<a href="https://test.salesforce.com/packaging/installPackage.apexp?p0=04t7F000000ZeWc" target="_blank">
  Deploy to Sandbox
</a>

### Implement your own user manager `Authen.WechatUserManagerBase`
* implement `loginSiteUserCore(sObject siteUser)`
* call `loginSiteUser()` to login user
```APEX
public class WechatUserManager extends Authen.WechatUserManagerBase {

	public WechatUserManager(FW.ISession2 session) {

		super(session);
	}

	public override sObject getSiteUserById(Id userId) {

		Contact appUser;

		if (String.isNotBlank(userId)) {
			contact[] appUsers = [select id from contact where id = :userId];

			if (appUsers.size() > 0) {

				appUser = appUsers.get(0);
			}
		}
		return appUser;
	}

	public override Id loginSiteUserCore(sObject siteUser) {

		return siteUser.Id;
	}

	public override sObject getSiteUserByWechatOpenId(string openId) {

		contact appUser;

		contact[] appUsers = [select id, wechatOpenId__c from contact where wechatOpenId__c = :openId];

		if (appUsers.size() > 0) {

			appUser = appUsers.get(0);
		}

		return appUser;
	}
}
```
### Implement `Authen.WechatIdentityController`
```APEX
public class WechatIdentityController extends Authen.WechatIdentityController {

	public override Authen.WechatUserManagerBase getWechatUserManager(FW.ISession2 session) {

		return new WechatUserManager(session);
	}

	public override Authen.WechatOptions getWechatOptions() {

		Authen.WechatOptions options = new Authen.WechatOptions();

		options.LoginPath = 'Login';
		options.LogoutPath = 'Logout';

		options.appId = 'wx1234567890123a9c';
		options.appSecret = 'a6b5069a12345678901234567890de25';

		return options;
	}
}
```
### set page action `Authen.WechatIdentityController.Authorize()`
```HTML
<apex:page controller="WechatIdentityController" action="{!Authorize}" doctype="html-5.0" applyhtmltag="false" applybodytag="false"
           showheader="false" sidebar="false" standardstylesheets="false">
```
### Authentication in custom REST API
```APEX
FW.ISession2 session = new FW.RestSession(RestContext.request, RestContext.response);
Authen.WechatRestAuthenticator authenticator = new Authen.WechatRestAuthenticator(RestContext.request, RestContext.response, new WechatUserManager(session));
sObject appUser = authenticator.getSiteUser();
string openId = authenticator.getOpenId();
```
### AJAX to invoke REST service
* HTTP header 'Site.Session' need to be set before send ajax request with the value of `document.cookie`
```JAVASCRIPT
$("#click").on("click", function(){
    axios.get('https://*.csX.force.com/services/apexrest/svc/v1/', {
            method:'get',
            headers: { 'Site.Session': document.cookie }
        })
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
});
```
