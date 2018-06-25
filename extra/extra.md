Go back to your IBM Cloud Dashboard.

## 1. Create a Tone Analyzer Service
1. Click the menu in the top left corner, and click `Watson`
2. In the left pane, click `Browse Services` under Watson Services
3. Click `Tone Analyzer`
4. Give your service a name, e.g. `tone-analyzer-service`, and choose what space it should be in, e.g. `dev`
5. Select the `Lite` plan
6. Click `Create`


## 2. Create a Serverless Function
2. Click the menu in the top left corner, and click `Functions`
3. Click the `Start Creating`-button
4. Click `Create Action`
5. Give the action a name, e.g. `tone-analyzer`. You can leave Enclosing Package as is. **Important**: Choose `Node.js 6` as Runtime.
6. Click `Create`


## 3. Connect the service to the action
At this stage you also need to connect the service to the action. Open GitBash or Terminal. Make sure you've installed the IBM Cloud CLI from here: https://console.bluemix.net/docs/cli/index.html#overview

1. Log into IBM Cloud through `bx login`
2. Enter your username and password
3. Run the command `bx target -o <YOUR_ORG> -s <YOUR_SPACE>`
4. Run the command `bx wsk service bind SERVICE_TYPE ACTION_NAME --instance SERVICE_NAME `, example: `bx wsk service bind tone_analyzer tone-analyzer --instance tone-analyzer-service`. You should get a response similar to this:
```
Credentials 'Auto-generated service credentials' from 'tone_analyzer' service instance 'tone-analyzer-service' bound to 'tone-analyzer'.
```
See more: https://console.bluemix.net/docs/openwhisk/binding_services.html#binding_services 

## 4. Finish creating and test your action
1. Go back to Menu > Function
2. Click Actions in the left pane
3. Click your action, e.g. `tone-analyzer`
4. Make sure you're in the code view, if not press `Code` in the left pane
5. Paste the code from here: 
```
var ToneAnalyzerV3 = require('watson-developer-cloud/tone-analyzer/v3');

function main(payload) {

    var toneAnalyzer = new ToneAnalyzerV3({
        username: payload.__bx_creds.tone_analyzer.username,
        password: payload.__bx_creds.tone_analyzer.password,
        version: 'v3',
        version_date: '2017-09-21'
    });
    
    return new Promise( (resolve, reject) => {

        toneAnalyzer.tone({text: payload.userInput}, (err, tone) => {
            if(err) 
                return reject({error: 'Error:' + err});
                
            return resolve({processedInput: tone});
        });

    });
}

exports.main = main;
```
6. Click `Save` in the top right corner and test the action by clicking Invoke. You will most likely get an error at this point, as you don't have any input specified.
7. Click `Change Input` and paste the following: 
```
{"userInput": "This is so stupid and useless"}
```
8. Click `Apply` and re-test the action by invoking it.
9. Take note of action username and password through going to Endpoints in the left pane, click the API-KEY link under REST API, and copy the key on the right. Username is before `:`, and password is after. Paste it in a note or similar, you will need this for the next step.
10. Take note of your spacename (or organization name + space). This is shown on the left side of where you got the username and password.

## 5. Connect the action to the Watson Assistant Dialog
1. Go back to your Watson Assistant dialog.  
2. Open the node that you want to invoke the action
3. Click the three dots just under "If bot recognizes" and click `Open JSON Editor`.
4. Edit (or add) the `context` property to the following:
```
"context": {
    "payload": "payload",
    "private": {
      "creds": {
        "user": "ACTION_USER_NAME",
        "password": "ACTION_PASSWORD"
      }
    }
  },
``` 
5. Under the `output` property, paste the following:
```
"actions": [
    {
      "name": "/<YOUR ORG>_<YOUR SPACE>/<YOUR ACTION NAME>",
      "type": "server",
      "parameters": {
        "userInput": "<?input.text?>"
      },
      "credentials": "<? $private.creds ?>",
      "result_variable": "context.payload"
    }
  ]
```
5. The result of your action's output can be viewed in the dialog node by writing `<? context.payload.processedInput ?>` or shorthand `$payload.processedInput` as a response. If you get an error having it as response, click the `Manage Context` link in the top right and see the result.