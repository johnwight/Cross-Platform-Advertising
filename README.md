# Cross-Platform-Advertising
This solution identifies cart abandonment activity (even for anonymous users) and subsequently targets the user with advertisements on different websites.

These instructions describe how to to implement cross-platform advertising with Adobe Experience Cloud and Adobe I/O products. The solution uses the following Adobe products:


1. [Introduction](#Introduction)

1. [Set Up Products](#Set-Up-Products)

1. [Watch the Solution Work](#Watch-It-Work)


## <a name="Introduction">Introduction</a>


### Solution Use Case

An anonymous user visits a retail website. The user visits the product page, selects a color and size of the product and adds the product into the cart. The user proceeds to checkout but before completing the transaction, leaves the website, and abandons the cart. The cart abandonment activity is captured in the Adobe Audience Manager segment and the user profile is served on multiple platforms to  provide relevant advertisements on different websites.

This solution features the unique value of identifying cart abandonment activity and subsequent advertising even with anonymous users. Though the solution works for authenticated users as well, this solution focuses on the anonymous user.


### Solution Architecture

The following diagram shows the architecture for this solution:

![advertising use case architecture](https://user-images.githubusercontent.com/29133525/35827000-c186acf4-0a77-11e8-9f96-a7522795df5d.jpeg)

### What's Needed

To complete this solution, you will need authorization to use the following services:

* Adobe Launch
* Adobe Analytics/Triggers
* Adobe I/O Events
* Adobe I/O Runtime
* Adobe Audience Manager APIs
* Adobe Audience Manager
* Third-party website for demonstrating solution

## <a name="Set-Up-Products">Set Up Products</a>

To set up Adobe products for this solution:

1. [Set Up Adobe Launch](#Set-up-Launch)

1. [Set Up Analytics Triggers](#Set-up-Triggers)

1. [Set Up Adobe I/O Runtime](#Set-up-Runtime)

1. [Set Up Adobe I/O Events](#Set-up-Events)

1. [Set Up Adobe Audience Manager](#Set-up-AM)

1. [Third Party Website](#Third-party)



To set up Launch:

1. On www.launch.adobe.com, click **New Property**.

   ![create new property](https://user-images.githubusercontent.com/29133525/34681760-13437952-f45a-11e7-898c-1561265c27b7.png)

1. On the **Create Property** box, provide the details for the new property and click **Save.**

   ![create property box details](https://user-images.githubusercontent.com/29133525/34681834-4af921e4-f45a-11e7-9f8f-3daca980310c.png)

1. Click the **Extensions** tab and install the Analytics extension. Add the Report Suite for **Development**, **Staging**, and **Production**. The full Report Suite ID is available at **Analytics** > **Admin** > **Report Suites**.

   ![analytics extension and report suite](https://user-images.githubusercontent.com/29133525/35827338-d4162dd0-0a78-11e8-9f9a-c54d30cc7644.png)

1. Add the **Experience Cloud ID Service** and **Adobe ContextHub** extensions with default settings and Click **Save**. 


   ![final extensions](https://user-images.githubusercontent.com/29133525/35827675-2b92d012-0a7a-11e8-95db-ddc78760cfb6.png)


1. Click the **Rules** tab and create a **Analytics** rule with the following specifications:

  
   ![analytics rule](https://user-images.githubusercontent.com/29133525/34682376-038da364-f45c-11e7-959b-413760b2c451.png)

1. For the **Adobe Analytics - Set Variables** action, click the **</> Open Editor** button at the bottom of the page and add the following custom script:

```
var cart=ContextHub.getItem("cart");
 
var price=[];
var items=[];
 
for(i=0;i<cart.entries.length;i++)
{
     price[i]=cart.entries[i].price;
     items[i]=cart.entries[i].title;
}
 
s.eVar4=price.join("|");
s.eVar5=items.join("|");
 
s.eVar8=_satellite.getVisitorId().getAudienceManagerLocationHint();
```


7. Click the **Environments** tab and create the following environments:

   * Dev
   * Stage
   * Production

      ![create environments](https://user-images.githubusercontent.com/29133525/34682566-7aee09b2-f45c-11e7-937a-0c5aadb0aef8.png)

8. Save the rule and then click the **Publishing** tab.

9. Click **Add New Library** button.

      ![add new library](https://user-images.githubusercontent.com/29133525/34682601-9a4a5860-f45c-11e7-9c40-6c7d89b20b61.png)
   
10. On the **Create New Library** form, specify a **Name** for the build and then select **Dev (development)** from the **Environment** drop down.

11. Click the **Add All Changed Resources** button. 
   
      ![specify build](https://user-images.githubusercontent.com/29133525/34682632-bb011508-f45c-11e7-98ef-7c7a55ebe898.png)
   
12. Under **Development**, select **Build for Development** in the library drop down.
   
      ![build for dev](https://user-images.githubusercontent.com/29133525/34682676-d8ac25c0-f45c-11e7-9987-2f79149b56fa.png)

13. Approve and publish the library by selecting the appropriate option under the drop down arrow for each phase of the build workflow (**Submitted** and **Approved**).

      ![full flow](https://user-images.githubusercontent.com/29133525/34682729-038d0f8e-f45d-11e7-927b-b730a66a802d.png)

14. Repeat this process for the **Stage** and **Production** environments as well.

15. The last step in the workflow is to select **Build and Publish to Production** on the drop down arrow under **Approved**.

      ![build and publish to production](https://user-images.githubusercontent.com/29133525/34682768-1f317504-f45d-11e7-819c-d29787ed322d.png)
   
### <a name="Set-Up-Analytics-Triggers">Set Up Analytics Triggers</a>

Triggers is a Marketing Cloud Activation core service that enables marketers to identify, define, and monitor key consumer behaviors, and then generate cross-solution communication to re-engage visitors. You can use triggers in real-time decisions and personalization.

For instructions on setting up Analytics Triggers, see [Specifying a New Trigger](https://github.com/adobeio/analytics-triggers-documentation#Specify-a-New-Trigger).

To set up Triggers for this solution:

1. Create a cart abandonment Triggers rule.

     ![cart abandonment rule](https://user-images.githubusercontent.com/29133525/34683199-87de61b0-f45e-11e7-976c-85cb99484ffa.png)

1. Add the following dimensions:

   * Custom eVar 3
   * Custom eVar 4
   * Custom eVar 5
   * Custom eVar 6
   * Custom eVar 7
   * Custom eVar 8
   * Page URL

1. To enable these dimensions in your report suite:

   On Analytics **Admin** screen, click **Report Suites**.

   1. Select a report suite and then select **Edit Settings** > **Conversion** > **Conversion Variables**.
   
      ![conversion variables](https://user-images.githubusercontent.com/29133525/34684000-f458eb2e-f460-11e7-9660-b7422735772c.png)


   1. Click **Add New** and then select the **Status** checkbox. Select **Enabled** from the drop-down menu and then click **Save**.

      ![status enabled](https://user-images.githubusercontent.com/29133525/34684251-bbfa042e-f461-11e7-9d19-8828414e9f8d.png)


   1. Add as many conversion variables as you need. It may take some time for them to appear on the Trigger dimensions screen.

 

### <a name="Set-up-Runtime">Set Up Adobe I/O Runtime</a>

In this solution, you can deploy the following script to handle Triggers I/O Events and to provide updates to the visitor profile in **Customer Attributes** via Target Profile APIs.

**webhook.js**
```
var request = require('request');
 
function main(args) {
 
  var method = args.__ow_method;
 
  if (method == "get") {
    var res = args.__ow_query.split("challenge=");
    var challenge = res[1];
    if (challenge)
    console.log("got challenge: " + challenge);
    else
    console.log("no challenge");
 
    return {
      body: challenge
    }
  }
 
    if (method == "post") {
      try{
        var body = new Buffer(args.__ow_body, 'base64');
        console.log("Message Body:"+body);
        var jSon = JSON.parse(body);
        var mcId = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].mcId;
        var index = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar5.data.length - 1;
        var trPrices = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar4.data[index];
        var trProducts = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar5.data[index];
        var dcs_region = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar8.data[index];
 
 
        var url = 'http://adobeiosolutionsdemo.demdex.net/event?trProducts='+trProducts+'&trPrices='+trPrices+'&d_mid='+mcId+'&d_orgid=C74F69D7594880280A495D09@AdobeOrg&d_rtbd=json&d_jsonv=1&dcs_region='+dcs_region;
        console.log("Calling AAM API with:"+url);
        return new Promise(function(resolve, reject) {
            request.get(url, function(error, response, body) {
                if (error) {
                    reject(error);
                }
                else {
                    resolve({body: response});
                }
            });
        });
      }catch(e){
        console.log("Error occured while calling the API",e);
      }
 
    }
 
}
```

To deploy the webhook.js and create a web action, enter the following commands:

```
wsk action create aam webhook.js
```
 
```
wsk action update aam --web raw
```


After entering the commands, the web action is accessible at the following location:

```
https://runtime-preview.adobe.io/api/v1/web/<your_openwhisk_namespace>/default/aam
```


### <a name="Set-up-Events">Set Up Adobe I/O Events</a>

To set up Adobe I/O Events:

1. After signing in to the [Adobe I/O Console](https://adobe.io/console), click **New Integration**.

    ![new integration button](https://user-images.githubusercontent.com/29133525/30292388-2ccdd986-96f3-11e7-93bd-93f74bb4e3a4.png)

2. Select **Receive real-time events** and click **Continue**.

    ![real time events](https://user-images.githubusercontent.com/29133525/30292486-946dc812-96f3-11e7-9bc5-2aa0196f704b.png)


3. Select **Analytics Triggers** as an event provider and click **Continue**.

    ![create io trigger](https://user-images.githubusercontent.com/29133525/30292595-064305ec-96f4-11e7-9e60-3ee811c7949a.png)

4. Click **Continue** to move on to the next page without making any changes.

5. Provide the **Name** and **Description** for your integration.

    ![name integration box](https://user-images.githubusercontent.com/29133525/30292714-6d00a8ac-96f4-11e7-8449-f93bb2fccfcb.png)

6. Generate a public certificate. To do this:

    1. Open a terminal and execute the following command:

        ```
        openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate_pub.crt
        ```
    2. Upload the public certificate by clicking the **Select a File** link and then by selecting the certificate from your computer:

        ![public certificate](https://user-images.githubusercontent.com/29133525/32476036-2e563e40-c332-11e7-9fbd-889f1ba8054f.png)

7. On the **Webhook Details** form, add details, including your web action URL for I/O Events integration webhook. This is where Triggers messages will be delivered by I/O Events. Click **Save**.

   ![webhook details](https://user-images.githubusercontent.com/29133525/34685236-efce0cca-f464-11e7-902a-27e594a82c24.png)

    For more information on creating and registering webhooks, see [Introduction to Webhooks](https://github.com/adobeio/adobeio-events-documentation/blob/master/Webhook_docs_intro.md).


### <a name="Set-up-AM">Set Up Adobe Audience Manager</a>

Adobe Audience Manager is a data management platform (DMP) that helps you build unique audience profiles so you can identify your most valuable segments and use them across any digital channel.

To set up Audience Manager:

1. In Audience Manager, click **Manage Data**.

   ![manage data](https://user-images.githubusercontent.com/29133525/34685969-4a38be56-f467-11e7-825b-ea138ea1b0ee.png)


1. Click **Traits**.

   ![traits](https://user-images.githubusercontent.com/29133525/34686031-8b31008a-f467-11e7-819e-587f67d9a769.png)



1. Create a new **Trait** with the **Expression Builder**, as shown:

   ![expression builder](https://user-images.githubusercontent.com/29133525/34685986-670e4190-f467-11e7-83be-223c0f8a9858.png)

   ![expression built](https://user-images.githubusercontent.com/29133525/34686086-bf416ed2-f467-11e7-992c-3c43288473d2.png)


1. Click **Segments** and create a new **Segment** as shown below. Make sure you choose the correct report suite. Select the trait that you created in the previous step.

   ![trigger segment](https://user-images.githubusercontent.com/29133525/34686347-7c194fca-f468-11e7-96ff-b0ca4d314e97.png)


1. Click **Destinations** and create a new destination as shown below.

   ![destinations](https://user-images.githubusercontent.com/29133525/34686733-bb9907b6-f469-11e7-8818-0412b4627bff.png)

1. Choose the **Browser** platform. 

1. Add the segment created in the previous step in segment mapping.

1. Save the changes.


### <a name="Third-party">Third-party Website</a>

For our demo, we will be deploying a sample website on different domain to retrieve the audience segment for the visitor and based on that we will be delivering a relevant advertisement for the visitor.

Create an ad banner specific for a marketing campaign targeted to specific audience segment. For example in our case we are interested in showing the below advertisement to an audience segment who added coats/jackets into the cart but later abandoned the cart. Needless to say that this audience segment has high potential customers who can eventually make a purchase (Because we know that they abandoned the cart with coat/jackets vs people who just visited coats/jackets products) and hence retailers would be ready to pay more money for serving the ad to this audience segment.



Add below script in the HTML <head> section of the webpage where you want to display the ad.


<head>
<script>
var demdex_seg="", jacketsCampaign=false;
function aam_cb() {
    if (typeof (arguments[0].stuff) != "undefined" && arguments[0].stuff != "") {
        demdex_seg = arguments[0].stuff;
        for(var i=0;i<demdex_seg.length;i++)
        {
            if(demdex_seg[i].cn=="trigger-aam")
            {
                jacketsCampaign=true;
            }
        }
    }
}
</script>
<script type="text/JavaScript" src="https://adobeiosolutionsdemo.demdex.net/event?d_stuff=1&d_dst=1&d_rtbd=json&d_cts=1&d_cb=aam_cb"></script>
</head>
Put the below script at page-bottom (just before </body> tag) to load the custom advertisement banner if the visitor is part of "trigger-aam" audience segment.
Ad placehoder in a webpage for reference:
<div class="ad_banner"><a id="ad_dest" href="#"><img id="aam_ad" src="images/addbanner_728x90_V1.jpg" alt=""></a></div>

changeAd
<script>
      if(jacketsCampaign)
           {
               document.getElementById("aam_ad").src="images/ad.jpg";  
               document.getElementById("ad_dest").href="http://localhost:4502/content/we-retail/us/en/products/men.html";
           }
    </script>
</body>
 
































Step 1: Adobe Launch
Adobe Launch is the next generation of Dynamic Tag Management. It provides a platform-based approach to building DTM extensions and a streamlined distribution system to quickly deploy client-side DTM libraries. Custom resources can now be created and reused within DTM to simplify the distribution of client side web applications.

Go to launch.adobe.com
Create new "Property"



Enter "Property" details



Go to Extensions→Catalog and install Adobe Analytics extension. Enter the Adobe Analytics report suite manually in the textbox.
Note: You can get the full report suite id in Adobe Analytics->Admin→Report Suites



Add Experience Cloud ID Service and Adobe ContextHub extensions with default settings and click save. Your final extensions page should look like this.



Create an Analytics rule defined as below.



In the "Adobe Analytics-Set Variables" action, go to the bottom of the page and add custom script in </> Open Editor as below:


1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
var cart=ContextHub.getItem("cart");
 
var price=[];
var items=[];
 
for(i=0;i<cart.entries.length;i++)
{
     price[i]=cart.entries[i].price;
     items[i]=cart.entries[i].title;
}
 
s.eVar4=price.join("|");
s.eVar5=items.join("|");
 
s.eVar8=_satellite.getVisitorId().getAudienceManagerLocationHint();



Go to "Environments" and create Dev, Stage and Production environment.




Save Rule and go to "Publishing", Click on "Add New Library".




Give name for the build, select Dev (Development) environment and then click "Add all changed resources".




Build for development and staging and approve for production.









Step 2: Analytics Triggers
Triggers is a Marketing Cloud Activation core service that enable marketers to identify, define, and monitor key consumer behaviors, and then generate cross-solution communication to re-engage visitors. You can use triggers in real-time decisions and personalization.

New to Triggers? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step5:Analytics/TriggersSetup

Create a simple cart abandonment Trigger rule as shown below.



Add below dimensions with Trigger.
Dimensions
 
Custom eVar 3
Custom eVar 4
Custom eVar 5
Custom eVar 6
Custom eVar 7
Custom eVar 8
Page URL
Note: To enables these dimensions in your report suite go to Adobe Analytics->Admin→Report Suites and follow below steps.

i) Select a report suite then click on Edit Settings->Conversion→Conversion Variables



ii) Click on Add New→check the status check box and select "Enabled" from the drop down menu. Click Save.



iii) Add as many conversion variables you need, it may take some time to show up in the Trigger dimensions UI.



Step 3: Adobe I/O Runtime
The Adobe I/O Runtime is a serverless platform that allows you to quickly deploy custom code to respond to events, execute functions right in the cloud, all with no server set-up.

For our demo we will be deploying a script for handling Triggers I/O Events and updating the visitor profile in Customer Attributes via Target Profile APIs.


webhook.js
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
var request = require('request');
 
function main(args) {
 
  var method = args.__ow_method;
 
  if (method == "get") {
    var res = args.__ow_query.split("challenge=");
    var challenge = res[1];
    if (challenge)
    console.log("got challenge: " + challenge);
    else
    console.log("no challenge");
 
    return {
      body: challenge
    }
  }
 
    if (method == "post") {
      try{
        var body = new Buffer(args.__ow_body, 'base64');
        console.log("Message Body:"+body);
        var jSon = JSON.parse(body);
        var mcId = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].mcId;
        var index = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar5.data.length - 1;
        var trPrices = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar4.data[index];
        var trProducts = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar5.data[index];
        var dcs_region = jSon.event["com.adobe.mcloud.pipeline.pipelineMessage"]["com.adobe.mcloud.protocol.trigger"].enrichments.analyticsHitSummary.dimensions.eVar8.data[index];
 
 
        var url = 'http://<your_aam_tenant>.demdex.net/event?trProducts='+trProducts+'&trPrices='+trPrices+'&d_mid='+mcId+'&d_orgid=<your_org_id>@AdobeOrg&d_rtbd=json&d_jsonv=1&dcs_region='+dcs_region;
        console.log("Calling AAM API with:"+url);
        return new Promise(function(resolve, reject) {
            request.get(url, function(error, response, body) {
                if (error) {
                    reject(error);
                }
                else {
                    resolve({body: response});
                }
            });
        });
      }catch(e){
        console.log("Error occured while calling the API",e);
      }
 
    }
 
}



Execute below commands to deploy the webhook.js and create a web action.


wsk action create aam webhook.js
 
wsk action update aam --web raw



After successful execution of the above commands, the web action is accessible in WWW at below location.


https://runtime-preview.adobe.io/api/v1/web/<your_openwhisk_namespace>/default/aam



Step 4: Adobe I/O Events
With Adobe I/O Events, you can code event-driven experiences, applications, and custom workflows that leverage and combine the Adobe Experience Cloud, Creative Cloud, and Document Cloud.

New to I/O Events? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step6:AdobeI/OConsoleIntegration

Provide your web action URL for I/O Events integration webhook, this is where Trigger messages will be delivered by I/O Events.

 


Step 5: Adobe Audience Manager
It’s a data management platform (DMP) that helps you build unique audience profiles so you can identify your most valuable segments and use them across any digital channel.

Go to Adobe Audience Manager.

Click on "Manage Data" on the left panel.



Click on Traits.



Create a new "Trait" with below expression.




Click on "Segments" and create a new "Segment" as below. Make sure you choose the correct report suite. Select the trait created in the above step.



Click on "Destinations" and create a new "destination" as below. Choose "Browser" platform and add the segment created in the above step in segment mapping.



Save the changes.


Step 6: Third Party Websites
For our demo, we will be deploying a sample website on different domain to retrieve the audience segment for the visitor and based on that we will be delivering a relevant advertisement for the visitor.

Create an ad banner specific for a marketing campaign targeted to specific audience segment. For example in our case we are interested in showing the below advertisement to an audience segment who added coats/jackets into the cart but later abandoned the cart. Needless to say that this audience segment has high potential customers who can eventually make a purchase (Because we know that they abandoned the cart with coat/jackets vs people who just visited coats/jackets products) and hence retailers would be ready to pay more money for serving the ad to this audience segment.



Add below script in the HTML <head> section of the webpage where you want to display the ad.

webpage
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
<head>
<script>
var demdex_seg="", jacketsCampaign=false;
function aam_cb() {
    if (typeof (arguments[0].stuff) != "undefined" && arguments[0].stuff != "") {
        demdex_seg = arguments[0].stuff;
        for(var i=0;i<demdex_seg.length;i++)
        {
            if(demdex_seg[i].cn=="trigger-aam")
            {
                jacketsCampaign=true;
            }
        }
    }
}
</script>
<script type="text/JavaScript" src="https://adobeiosolutionsdemo.demdex.net/event?d_stuff=1&d_dst=1&d_rtbd=json&d_cts=1&d_cb=aam_cb"></script>
</head>
Put the below script at page-bottom (just before </body> tag) to load the custom advertisement banner if the visitor is part of "trigger-aam" audience segment.
Ad placehoder in a webpage for reference:
<div class="ad_banner"><a id="ad_dest" href="#"><img id="aam_ad" src="images/addbanner_728x90_V1.jpg" alt=""></a></div>

changeAd
<script>
      if(jacketsCampaign)
           {
               document.getElementById("aam_ad").src="images/ad.jpg";  
               document.getElementById("ad_dest").href="http://localhost:4502/content/we-retail/us/en/products/men.html";
           }
    </script>
</body>


Step 7: Let's Test 
Execute below steps to test and debug the solution.

Go to https://aam-demo2.herokuapp.com as you can see that currently there are "No ads to show".



Go to http://localhost:4502/content/we-retail/us/en/products/men.html

Click on any jacket or coat you like.



Select a Color and Size and click on "ADD TO CART".



Click on Checkout.



Enter details and click continue.



Close the tab to initiate the cart abandonment scenario.

Go to Triggers UI page, wait for your trigger event to surface.



Go to Command Line Interface (CLI) and execute below commands.

wsk activation list aam



Select the top most activation and execute below command.

wsk activation get f6f5ae1dcb3d4292991d63f22283fb94



Go to https://aam-demo2.herokuapp.com/index.html again and you will see that the ad banner is updated. A custom ad banner will be displayed to visitor!

