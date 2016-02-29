#Realeyes client side javascript data collection api

There are many ways to integrate Realeyes emotion measurement with various survey software or any other web environment. In this documentation we will describe these different approaches.

### Realeyesit.IntegrationAPI components
Realeyesit.IntegrationAPI contains the following components:
- *Realeyesit.IntegrationAPI.Client* - API for working with Realeyes emotion measurement component
- *Realeyesit.IntegrationAPI.IntegrationManager* - Script to organize communications in case cross-domain integrations are used

### High level description of the integration:
Usually our solution is integrated to the sites with the following structure:
- main window (contains the frameset)e
    - child frame with clients survey content
    - child frame with realeyes emotion measurement component

Main idea of the integration is that the survey software shows their content with usual flow and only when media playback with emotion measurement is needed, the software will send commands to Realeyes API via the Client (commands like: askPermissions, preloadMedia, play etc.) which in turn then displays the neccesary webcam prompts and media.

The frame approach is required because of the security requirements of webcam access. Namely the access to the webcam can only be requested for the duration of a single page and in case the survey consists of multiple pages, then it would mean bothering the user multiple times. Having it in a frame that's never loaded, would work around that.

To make the integration work, all components should be placed correctly:

Integration manager should be included in the main window.

````html
<head>
  <script src="//codesdwncdn.realeyesit.com/IntegrationAPI/latest/Realeyesit.IntegrationAPI.IntegrationManager.js"></script>
</head>
 ````

Integration client should be included in the child iframe with survey content.
````html
<head>
  <script src="//codesdwncdn.realeyesit.com/IntegrationAPI/latest/Realeyesit.IntegrationAPI.Client.js"></script>
</head>
 ````

The iframe with realeyes emotion measurement component should be added on page and src should have link like:

````html
<frame id="dataCollectionIframe" name="dataCollectionIframe"
       src="https://collect.realeyesit.com/A807JJ?jsapienabled=true"/>
````

Obligatory attributes of the Iframe.
- **ID** should be set to datacollectionIframe, so that our manager script can find it.
- **Src** should be set with the following attributes
  - The **baseUrl** of the collection site (eg. http://stagecollect.realeyesit.com (our sandbox), http://collect.realeyesit.com (live platform))  
  - The **JsApiEnabled** query string parameter, you have to set it true, thus you can communicate with our javascript API.
  - The **isReviewer** query string parameter, if true then session will be excluded from analysis.
  - The **CustomParticipantID** query string parameter, you should put your session\participant identity to the parameter.
  - (optional) The **URL hash** of the involved collection (eg. A8077J)
*NOTE:* Be aware of the parent framesetsâ€™ rows/cols attribute, you might want to modify it after adding frames.

 ````html
 <frameset id="frameset" rows="*,1,1" border="0" frameborder="no" framespacing="0" noresize="">
 ````
 *Note:* Pay attention not to reload our emotion measurement component iframe, because the webcam access will be lost that way and the user would need to be prompted for access again.

When all the component have been added and loaded then the init method should be called as below:
````javascript
Realeyesit.IntegrationAPI.IntegrationManager.init(window.document.getElementById('dataCollectionIframe'));
````

### IntegrationManager.onReady

**onReady**. Indicates that all API related stuff was downloaded and initialized and now ready to be used. Has 1 callback parameter. the callback will be executed when manager will be ready.


##Collection set-up description

There are 2 ways to define which media content should be displayed when measuring emotions:
- setup collection manually via our delivery portal: http://www.realeyesit.com/get-started
- setup all data via integration configuration programmatically  

### Manual collection setup
You can read more how to create collections via our portal on our site: http://www.realeyesit.com/get-started
After creation the collection you can use UrlHash of created collection and pass it as path parameter of the realeyes frame url (see previous example).

### Programmatic collection setup
With manual collection setup it'd take a lot of work to create and manage many collections at the same time. For larger volumes we've built the programmatic collection setup api.

To get started with programmatic API you need to create yourself an API key by which we'd identify all the data that's being collected.  Follow these steps below to get started:
* Log into the delivery site at http://delivery.realeyesit.com with a user that has admin rights
* Go to Account from the main menu
* Go to API Keys from the sub-menu
* Pick the account you want to create API keys to
* Under the "Rest API Keys" click create

Note that in case of using programmatic collection setup then you need to take care of properly encoding and distributing the media for playback on your own. When you manually setup collections, Realeyes encodes videos to multiple sizes for optimal playback automatically and distributes them across Amazon Cloudfront CDN, but that's not the case when using the programmatic api.

To use the api, then instead of manually creating the collection and passing the url hash to the api, you can pass a configuration to the init method as shown below:

````javascript
var frame=window.document.getElementById('dataCollectionIframe');
var config = {
    apiKey:"3c2819e2026d6168",
    key:"myFirstCollection",
    name:"Test Collection via config",
    displayLang:"1043",
    translations:[{key:"dc_allow_webcam_instruction",value:"my new <b>overriden</b> instruction"}],
    startOptions: {qualityFeedbackType: 1, externalCssUrl: 'http://localhost:8080/public/assets/css/test.css'},
    skinName:"v2",
    alternativeFlashCNAME:"adjustedCname",
    elements:[
        {key:"video1" ,url:"https://mysite/myvideo1.mp4"}]
    };
Realeyesit.IntegrationAPI.IntegrationManager.init(frame, config);
````
Where config object has next attributes:

- apiKey - key to identify the account under which all the collected data would belong to, the same key used with all other server side API calls as well
- key - unique external key identifying the test or the project. **IMPORTANT**: each porject should have 1 unique key. There are shouldn't be 2 and more projects with same key and one project shouldn't have different keys! 
- name (optional) - a friendly name of the test or project to be used when reporting the results, the key will be used if the name is empty
- displayLang (optional) - the language of all the user interfaces and everything else. Default would be English
- translations (optional) - the array of ue messages which should be overriden with new text.
    - key - key of ui message. The kye can be retrived from ui. It placed to data-ui-message-key attribute of html element
    - value - new ui message which will be applied
- startOptions (optional) - start options for UI
    - qualityFeedbackType - type of of quality feedback screen (1 - auto, 2 - manual, 3 - skip, any other - default)
    - externalCssUrl - css file which should be applied to the UI skin
- ignoreTrackingErrors (optional) - the playback of video will be continued and any tracking errors/exception will be ignored
- skinName (optional) - name of skin which will be applied for all visible pages of Realeyes emotion measurement component. If the parameter is skipped the default skin will be used. Possible values: 'original', 'IPSOSConnectV7', 'IPSOSConnectV8'
- alternativeFlashCNAME (optional) - if Realeyesit company adjested for you CNAME then plese put alias to the filed if you want to show access permission dialog with adjusted domain url
- elements - array of media elements configurations:
    - key - unique external key identifying the media.  **IMPORTANT**: each media should have 1 unique key. There are shouldn't be 2 and more videos with same key and one video shouldn't have different keys! 
    - url - The URL of the media that would be played back. We support mp4 and flv. *NOTE:* Only full preload of the media from given URL is supported now. Media needs to be in the appropriate size for serving (APIâ€™s do not apply any conversions on video files for playback, so files should be appropriately presized). Be aware that proper crossdomain.xml file should be present at the url domain location for our flash player to download the media. See next section for details.
    - name (optional) - a friendly name of the media to be used in reporting, the key will be used if the name is empty
    - randomization (optional) - object with attributes 'elementsGroup' and 'randomisationGroup'. The object will be used to define randomisation rules for the playback.
    - skipRecording (optional) - if empty, media will be measured and billed, if set to true, then the media will be considered as **clutter** and will just be shown, but not measured and billed
    - testElementKey (optional) - unique external key identifying the exact element
- metadata(optional) - dictianary from with custom data which should be passed to RealeyesitAPI. Supported values
  - privacyLink(optional) - the link will replace privacy link on Realeyesit UI
  - faqLink(optional) - the link will replace link to FAQ on Realeyesit UI
  - supportLink(optional)- the link will replace link to Support on Realeyesit UI
  - progressValue(optional) - defines progress of whole survey (value from 0 to 100). Provide it if your skin contains progress bar and the progress bar should display global progress.

**NOTE**: the following predefined classes can be used to create the configuration:
- Realeyesit.IntegrationAPI.Configuration
- Realeyesit.IntegrationAPI.ConfigurationElement
- Realeyesit.IntegrationAPI.RandomisationDescription
example:

````javascript
       var frame=window.document.getElementById('dataCollectionIframe');
       var config = new Realeyesit.IntegrationAPI.Configuration();
       config.displayLang = "1043";
       config.apiKey = "3c2819e2026d6168";
       config.key = "testColl1";
       config.name = "Test Collection via config";

       // optional
       config.metadata = { privacyLink: '//yoursite.com/privacy', faqLink: '//yoursite.com/faq', supportLink: '//yoursite.com/support', progressValue: 73 };

       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "videoUniqueKey1";
       el1.name = "insurance via config";
       el1.url = "https://devsourcedatabucket.s3-eu-west-1.amazonaws.com/somevideo1.mp4";
       el1.testElementKey ="elementkey1";

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "videoUniqueKey2";
       el2.url = "https://devsourcedatabucket.s3-eu-west-1.amazonaws.com/somevideo2.mp4";

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "videoUniqueKey1";
       el3.url = "https://devsourcedatabucket.s3-eu-west-1.amazonaws.com/somevideo1.mp4";
       el3.testElementKey ="elementkey1";

       config.elements = [el1,el2,el3];

       Realeyesit.IntegrationAPI.IntegrationManager.init(frame, config);
````

 **IMPORTANT**: There are couple examples to understand the project's key and media's key handling:

*Example 1*
We have started our project with name "Project 1" and key "project1"

````javascript       
       ...
       var config = new Realeyesit.IntegrationAPI.Configuration();
       config.key = "project1";
       config.name = "Project 1";     
       ...  
      
````

After couple days (or even in same time) we have started our second project with name "Project 2" and same key "project1"

````javascript       
       ...
       var config = new Realeyesit.IntegrationAPI.Configuration();
       config.key = "project1";
       config.name = "Project 2";     
       ...        
````

In the result all data will be collected under "Project 1" and project with name "Project 2" won't be created in the system because we are handling it by project's key and it was same as before.

*Example 2*
We have started our project with name "Project 1" and key "project1"

````javascript       
       ...
       var config = new Realeyesit.IntegrationAPI.Configuration();
       config.key = "project1";
       config.name = "Project 1";     
       ...  
      
````

After couple days (or even in same time) we have desided to chande key for the project to "project2"

````javascript       
       ...
       var config = new Realeyesit.IntegrationAPI.Configuration();
       config.key = "project2";
       config.name = "Project 1";     
       ...        
````

In the result data will be collected under 2 different project but with same name "Project 1".


#### Crossdomain file loading

When using Programmatic collection setup, you can specify any url for the media to be downloaded and played in our flash player. Due to flash security policy our player can not download content from domain other than our, unless remote domain is not configured to allow this. For that purpose remote domains should host crossdomain.xml files (e.g. at the root 'somedomain.com/crossdomain.xml') that allow our player's domain to access this domain. Here is the example of such a file:
````xml
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM "http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
   <allow-access-from domain="*.realeyesit.com"/>
</cross-domain-policy>
````
You can read more on crossdomain policy files [here](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)
#### Randomisation settings
When showing multiple stimuli to viewers, it's important to try to minimize or eliminate any bias that might come from a specific ordered sequence of presenting the stimuli. This is where Randomization comes in.
A test consists of a sequence of stimuli which might be media or instructions.
Each stimulus in a sequence can belong to a group. A group is a collection of stimuli which one wants to treat as a single entity from the perspective of randomization. For example you might have an instruction that specifically refers to an ad, like "Watch this ad about lions" and then you have the ad that follows it which is about lions - you need these two to go together. Or if you have 2 ads among many other ads that always need to be shown in a specific order.
The order of stimuli inside a group is always maintained.
Each of the groups of stimuli can in turn be assigned to randomization groups. The order of each stimulus group inside the randomization group will be randomized.
We always record the randomization rules and actual outcome for each session separately in the database. The randomization rules will give us the likelihoods of seeing each combination and it's easy to verify if the randomization works by comparing the observed frequencies of combinations to the expected ones coming from the rules.

As you can see each media has randomization configuration element with 2 attributes:
- elementsGroup
- randomisationGroup

see examples to understand how the randomization configuration works:

Example 1: Let's assume I have 4 media and I want to each of them to be in random position with every test, the configuration would be following
````javascript
       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "video1";
       el1.url = "https://mysite/video1.mp4";
       el1.randomisation = {elementsGroup:'1'};

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "video2";
       el2.url = "https://mysite/video2.flv";
       el1.randomisation = {elementsGroup:'2'};

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "video3";
       el3.url = "https://mysite/video3.mp4";
       el1.randomisation = {elementsGroup:'3'};

       var el4 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el4.key = "video4";
       el4.url = "https://mysite/video4.flv";
       el1.randomisation = {elementsGroup:'4'};

       config.elements = [el1,el2,el3,el4];
````
Example 2: Let's assume we have 4 media, but we want the media 1 and 2 to be always one after the other and media 3 and 4 also one after the other, but the groups themselves would be in a different position each time.
````javascript
       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "video1";
       el1.url = "https://mysite/video1.mp4";
       el1.randomisation = {elementsGroup:'1'};

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "video2";
       el2.url = "https://mysite/video2.flv";
       el1.randomisation = {elementsGroup:'1'};

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "video3";
       el3.url = "https://mysite/video3.mp4";
       el1.randomisation = {elementsGroup:'2'};

       var el4 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el4.key = "video4";
       el4.url = "https://mysite/video4.flv";
       el1.randomisation = {elementsGroup:'2'};

       config.elements = [el1,el2,el3,el4];
````
Example 3: Let's assume we have 4 media, but we want the media 1 and 2 to be always one after the other and the rest to be randomized individually.
````javascript
       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "video1";
       el1.url = "https://mysite/video1.mp4";
       el1.randomisation = {elementsGroup:'1'};

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "video2";
       el2.url = "https://mysite/video2.flv";
       el1.randomisation = {elementsGroup:'1'};

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "video3";
       el3.url = "https://mysite/video3.mp4";
       el1.randomisation = {elementsGroup:'2'};

       var el4 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el4.key = "video4";
       el4.url = "https://mysite/video4.flv";
       el1.randomisation = {elementsGroup:'3'};

       config.elements = [el1,el2,el3,el4];
````

Example 4:Let's assume we have 4 media, but we want the media 3 to always stay where it is, but others to move around it randomly.
````javascript
       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "video1";
       el1.url = "https://mysite/video1.mp4";
       el1.randomisation = {elementsGroup:'1'};

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "video2";
       el2.url = "https://mysite/video2.flv";
       el1.randomisation = {elementsGroup:'2'};

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "video3";
       el3.url = "https://mysite/video3.mp4";

       var el4 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el4.key = "video4";
       el4.url = "https://mysite/video4.flv";
       el1.randomisation = {elementsGroup:'3'};

       config.elements = [el1,el2,el3,el4];
````
Example 5: Let's assume we have 4 media, but we want the media 1 and 4 to only be randomized between themselves and then media 2 and 3 to be randomized between themselves.
````javascript
       var el1 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el1.key = "video1";
       el1.url = "https://mysite/video1.mp4";
       el1.randomisation = {elementsGroup:'1',randomisationGroup:'1'};

       var el2 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el2.key = "video2";
       el2.url = "https://mysite/video2.flv";
       el1.randomisation = {elementsGroup:'2',randomisationGroup:'2'};

       var el3 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el3.key = "video3";
       el3.url = "https://mysite/video3.mp4";
       el1.randomisation = {elementsGroup:'3',randomisationGroup:'2'};

       var el4 = new Realeyesit.IntegrationAPI.ConfigurationElement();
       el4.key = "video4";
       el4.url = "https://mysite/video4.flv";
       el1.randomisation = {elementsGroup:'4',randomisationGroup:'1'};

       config.elements = [el1,el2,el3,el4];
````

##Description of control emotion measurement process.
Once you referenced the integrationmanager in your root html file, you can reference and use the integrationApiClient library in any of the child frames.
The library has the following public functions.
- **askWebcamPermissons**. Asks user for webcam access.
- **checkWebcamQuality**. Checks webcams and switchs to active webcam. Asks user to adjust environment if it needed.
- **preloadMedias**. Starts preloading the involved media.
- **play**. Start playing the involved media
- **addSessionStateListner**.  Register a function as a new callback for session notification messages
- **getSessionInfo**.  The method returns our internal sessionId and current session state
- **onReady**. Indicates that all API related stuff was downloaded and initialized and now ready to be used. Has 1 callback parameter which will be executed when client will be ready.

The first 3 methods have 3 callback parameters: success, failed, progress.

````javascript
var successCallback = function(msg) {
              console.log(msg);
};

var failedCallback = function(msg) {
              console.error(state);
};

var progressCallback = function(msg) {
              console.info(state);
};

Realeyesit.IntegrationAPI.Client.askWebcamPermissions(callback, failedCallback, progressCallback);
````
**Note:** if you execute **Realeyesit.IntegrationAPI.Client.play()** before **askWebcamPermissons** then we will not record participant's emotion data and will just play videos. To start emotion tracking you need execute **askWebcamPermissons** before **play** method.

### Realeyesit.IntegrationAPI.Client
#### onReady.

- **onReady**. Indicates that all API related stuff was downloaded and initialized and now ready to be used. Has 1 callback parameter. the callback will be executed when client will be ready.

#### askWebcamPermissons.

- **askWebcamPermissons**. Asks user for webcam access. Has 4 parameters:1 boolean *dontCheckCameraQuality* and 3 callback parameters, all of them will be called with one parameter, which contains two properties **state** and (optionally) **reason**.

    Possible values of the state property: readyToUse | allowed | denied | failed
    **NOTE:** state readyToUse will be returned only if parameter *dontCheckCameraQuality* = false, null or undefined. Otherwise the check web cam quality dialog will not be shown and final states for *askWebcamPermissons* will be: 'allowed' | 'denied'

````javascript
var successCallback = function(msg) {
              alert(msg.state);
};

var failedCallback = function(msg) {
              alert(msg.state);
};

var progressCallback = function(msg) {
              alert(msg.state);
};

//if want to ask webcam permissions and check quality of webcam immideatelly. Final states then readyToUse | denied | failed
Realeyesit.IntegrationAPI.Client.askWebcamPermissions(callback, failedCallback, progressCallback);

//or

//if want to ask webcam permissions. Final states then: allowed | denied | failed
Realeyesit.IntegrationAPI.Client.askWebcamPermissions(callback, failedCallback, progressCallback, true);
````
**NOTE:** if you set parameter *dontCheckCameraQuality* = true then you should execute *checkWebcamQuality* method before start playing medias. Otherwise it will be called automatically and check quality screen will be displayed 

#### checkWebcamQuality.

- **checkWebcamQuality**. Checks webcams and switchs to active webcam. Asks user to adjust environment if it needed.  Has 3 callback parameters, all of them will be called with one parameter, which contains two properties **state** and (optionally) **reason**.

    Possible values of the state property: readyToUse

````javascript
var successCallback = function(msg) {
              alert(msg.state);
};

var failedCallback = function(msg) {
              alert(msg.state);
};

var progressCallback = function(msg) {
              alert(msg.state);
};

Realeyesit.IntegrationAPI.Client.checkWebcamQuality(callback, failedCallback, progressCallback, dontCheckCameraQuality);
````

#### preloadMedias.

- **preloadMedias**. Starts preloading the involved media. Has 3 callback parameters.
   - progressCallback: called with one parameter, contains 2 property **state** and **progress**, the latter has the following properties


      - itemsCount
      - loadedBytes
      - remainingBytes
      - remainingTimeEstimate
      - speed
      - totalBytes

    Possible values of the state property: started

   - finishedCallback/failedCallback: called with one parameter, contains 2 properties **state** and **reason**.

    Possible values of the state property: finished

 ````javascript
 var progressCallback=function(msg){
    if(msg.status=='started'){
        alert('download started');
    }
    if(msg.status=='donwloading'){
        alert(msg.progress.remainingTimeEstimate);
    }
 }

 var finishedCallback=function(msg){
    alert(msg.state); //msg.state='finished'
 }

 var failedCallback=function(msg){
      alert('Downloamd failed');
 }

 Realeyesit.IntegrationAPI.Client.preloadMedias(finishedCallback, failedCallback, progressCallback);
 ````

#### play.

- **play**. Start playing the involved media. Has 3 callback parameters, all of them will be called with one parameter, which contains two property; **state** and **reason**. ProgressCallback contains an additional property  **element** that shows info about the current element like:
 * ExternalKey - key value of media from provided configuration
 * SequenceNo - sequence number of element during displaying
 * URL - url to media element
 * TestElementExternalKey - uniqe identity of test element (testElementKey from initial configuration)

    Possible values of the state property: started | playing | finish-playing | finished

````javascript
Realeyesit.IntegrationAPI.Client.play(succesfullCallback, failedCallback, progressCallback)
````
#### addSessionStateListner.

- **addSessionStateListner**.  Register a function as a new callback for session notification messages. Listener will be called with one parameter, which is a JSON object having 3 attributes
    - id: numeric representation of the state
    - name: name of the state
    - reason: complementary message to the state

 Possible values of the state:
       * id: 13, name: 'Collected'
       * id: 14, name: 'Declined',
       * id: 15, name: 'Failed'


````javascript
var messageHandler=function(msg){
    alert(msg.name);
}

Realeyesit.IntegrationAPI.Client.addSessionStateListner(messageHandler);
````


#### getSessionInfo.

- **getSessionInfo**. Returns to you object with current sessionId and state with the following object:

````javascript
{
sessionId,
stateId,
reasonId,
reasonName
}
````
there are list of all supported states:

*ID-Name*
* 1- New
* 2- Checking Environment
* 3- Selecting Webcam
* 4- Requesting Access to Webcam
* 5- Checking Video Quality
* 6- User is Correcting Environment
* 7- Requesting Full Screen Permission
* 8- Calibrating
* 9- Processing Calibration
* 10- Preloading Media
* 11- Collecting
* 12- Processing
* 13- Collected
* 14- Declined
* 15- Failed
* 16- Processed
* 17- Processing Failed
* 18- Checking Camera Feeds
* 19- Connecting to media server

**NOTE:** reasonId and reasonName it's autogenerated stuff and it can be just a text message which clarify why state was changed to current one.

Examples:
````javascript
 {sessionId: 905688, stateId: 10, reasonId: 103, reasonName: "Environment Checks Succeeded"}
 {sessionId: 905688, stateId: 19, reasonId: 128, reasonName: "Media preloaded"}
 {sessionId: 905688, stateId: 13, reasonId: 201, reasonName: "Success, study completed, results submitted"}
````

````javascript
Realeyesit.IntegrationAPI.Client.play(callback)
````


Example:
````javascript
<script>


                 var parentWin = parent;
                 if (window.addEventListener) {
                     window.addEventListener("load", function () { startFunction(); });
                 } else if (window.attachEvent) {
                     window.attachEvent("onload", function () { startFunction(); });
                 }
                 var startFunction = function () {
                     var firstscript = parentWin.document.getElementsByTagName('script')[0];
                     var script = document.createElement('script');
                     script.type = 'text/javascript';

                     if (script.addEventListener) {
                         script.addEventListener("load", function () { insertFrame(); });
                     } else if (script.attachEvent) {
                         script.attachEvent("onload", function () { insertFrame(); });
                     }

                     script.async = 1;

                     script.src = '//codesdwncdn.realeyesit.com/IntegrationAPI-Debug/all/latest/Realeyesit.IntegrationAPI.IntegrationManager.js';

                     firstscript.parentNode.insertBefore(script, firstscript);

                     //   head.appendChild(script);

                     var insertFrame = function () {
                         var newFrame = parentWin.document.createElement("frame");
                         newFrame.setAttribute("id", "dataCollectionIframe");
                         newFrame.setAttribute("name", "dataCollectionIframe");

                         newFrame.onload = function () {
                             parentWin.Realeyesit.IntegrationAPI.IntegrationManager.init(newFrame);
                             if (typeof callback === "function") {
                                 callback.apply(null, []);
                             }
                         };
                         newFrame.setAttribute("src", "//localhost:5006/kexbop?jsapienabled=true&origin=" + location.origin);
                         var mainFrame = parent.document.getElementById("mainFrame");
                         mainFrame.parentElement.rows = "*,1,1";
                         mainFrame.parentElement.appendChild(newFrame);
                     };
                 };
</script>

````

#### checkEnvironment.

- **checkEnvironment**. checks if user's environment is capable to run Realeyesit's software. Checks if browser is capable, flash version, webcam etc. There are two parameters for the method:   
    - succesfullCallback: register callback function which will be executed if environment is capable 
    - failedCallback: register callback function which will be executed if environment isn't capable
    
both callbacks will recieve next json object 
    
````javascript
//In case when all tests are passed
{
    checksPassed: true
    ,failureReasonCode: null
    ,failureReasonString: null
}
  
//In case when a check fails
{
    checksPassed: false
    ,failureReasonCode: 3
    ,failureReasonString: FLASH_BLACKLISTED
}
````
   
Possible failure reasons that we report
 * 1 - FLASH_NOT_INSTALLED
 * 2 - FLASH_TOO_OLD
 * 3 - FLASH_BLACKLISTED
 * 4 - BROWSER_BLACKLISTED
 * 5 - MOBILE_BROWSER
 * 6 - NO_WEBCAMS_DETECTED
 * 7 - OTHER_ERROR


example of usage: 

````javascript
var messageHandler=function(msg){
    alert(msg.name);
}

Realeyesit.IntegrationAPI.Client.checkEnvironment(messageHandler);
````

*NOTE*: All script files and other resources for the test execution are loaded over Amazon Web Services Cloudfront content delivery network with more than 35 edge locations across the world and are compressed to improve the user experience (except media files unless a CDN URL is given for them).
