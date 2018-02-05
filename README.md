# Cross-Platform-Advertising
Integrate cross platform advertising with Adobe I/O and Adobe Launch


Introduction
This blog is a step by step guide to implement the cross platform advertising solution using Adobe Experience Cloud and Adobe I/O products integration. The solution uses below Adobe products:

Adobe Launch (The next generation dynamic tag management platform)
Role: Adobe launch loads the Core, Analytics, Experience Cloud ID Service, Adobe ContextHub configuration and scripts on the client browser when the website page is requested.

Adobe Analytics/Triggers
Role: Analytics Triggers maintains the definition of rules that in which circumstances a trigger should be fired, for our case if a visitor adds a product into a cart but doesn't make the purchase a trigger message should get fired with visitor activity information.

Adobe I/O Events
Role: Adobe I/O Events is responsible for getting trigger event from pipeline services to your custom webhook endpoint. It can be managed in Adobe I/O Console (https://console.adobe.io) integration.

Adobe I/O Runtime
Role: Adobe I/O Runtime is our serverless platform where we have hosted a web action to perform some activity in response to I/O Events. It is referred as a custom webhook endpoint in Adobe I/O Console integration.

Adobe Audience Manager APIs
Role: Adobe Audience Manager Events DCS APIs are responsible for sending trait signals from I/O Runtime web actions. It records the Marketing cloud ID and updates the visitor profile based on trigger event.
 
Adobe audience Manager
Role: Adobe Audience Manager maintains the definitions of traits/segments and destinations.

Third party websites
Role: Demo websites which is customized to update the ad banners for specific audience segments. 


User Story:

"An anonymous user visits a retail website. The user is interested in buying a product. He/she visits the product page, selects a color and size of the product and adds the product into the cart. The user proceeds for checkout but before completing the transaction, user leaves the website abandoning the cart. The cart abandonment activity is captured in Adobe Audience Manager segment and the user profile is served on multiple platform to display relevant advertisements on various different websites."

Note: Though our solution is valid for both anonymous/authenticated user, the real value is in identifying the cart abandonment activity of an anonymous user and able to target them on different websites with the help of Experience Cloud ID Service, Analytics Triggers, Adobe I/O (Events & Runtime) and Adobe Audience Manager. Hence for this demo our focus is on anonymous user use case.

Architecture:






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

