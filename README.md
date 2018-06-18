# Workshop - Calling serverless functions from Watson Assistant 

## Lab 1: Create a Watson Assistant dialog

In this first lab, you'll create the base conversation for the workshop. We'll use the use case of booking a restaurant through a chatbot. 
- If you feel like creating your own conversation, start [**here - Step 1**](https://github.com/IBMCodeLondon/chatbot-workshop/blob/master/training.md#step-1-optional-designing-your-bot)
- If you'd rather follow an example go straight to [**Step 2**](https://github.com/IBMCodeLondon/chatbot-workshop/blob/master/training.md#step-2-train-watson-assistant-service).

Come back here when you're done!

Bravo! You've created a chatbot that pretends to make a reservation for you.
Let's now see how to create a serverless function that you'll be able to call from your workspace.

## Lab 2: Create a basic serverless function

You should now have a conversation workspace asking the user information about booking a table at a restaurant. If not, make sure that you've followed the first lab.
Following on this workshop you will see how to call serverless functions directly from the Watson Assistant service. 
This will allow the conversation to be driven by the service rather than having to create a middleware app that handles redirections to other services based on the intents detected.

**Apply a feature code for your account**
Before starting this lab, ask your instructor for a feature code that will unlock 

1. Go to your [IBM Cloud dashboard](https://console.bluemix.net/dashboard/apps)
2. Click on the hamburger menu on the top-left and select "Functions"
3. Select "Start Creating" then "Create action"
4. On the "Region" dropdown, select US-South
5. Give a name to your action (ie. "__book__") and create a new package for it (ie. "__bot__")
6. Choose your preferred language (the examples will be given in Node.js) then click Create
7. Your function is pre-created with a Hello World sample, replace the text by "Your table has been booked!", click "Save" then "Invoke" on the top-right corner to try it out

Congrats, you now have a serverless function that says your table has been booked... we're not quite there yet! 

## Lab 3: Call a serverless function from your Assistant

Now, you've got a bit of chatbot and a bit of serverless function... let's connect them together! 
![FUSION](https://github.com/IBMCodeLondon/chatbot-serverless-workshop/blob/master/giphy.gif?raw=true)

**Retrieve your Cloud Function service credentials**

1. On the "Functions" side-nav, under "Getting started", click on "CLI"
2. Write down your Namespace, showed in step 3, it will look like `youraccountname_dev` 
3. On the "Functions" side-nav, under "Getting started", click on "API Key"
4. Click the eye sign to display your Key, it will look like `a-long-username:acomplicatedpassword`
5. Write it down under the following form:
```json
{
  "user":"a-long-username",
  "password":"acomplicatedpassword"
}

```

**Call the function from Watson Assistant**

1. Head back to the Dialog tab of your chatbot workspace
2. Add your Cloud Function credentials as a context variable named `$private.my_credentials`:
![botzero](https://github.com/IBMCodeLondon/chatbot-serverless-workshop/blob/master/bot0.gif?raw=true)
3. Edit the response of the book_reservation node using the JSON editor
![botone](https://github.com/IBMCodeLondon/chatbot-serverless-workshop/blob/master/bot1.gif?raw=true)
4. Replace with the following code snippet:
```json
{
  "output": {
    "text": {
      "values": [
        "Great, let me do the booking for you..."
      ],
      "selection_policy": "sequential"
    }
  },
  "actions": [
    {
      "name": "/Namespace/PackageName/ActionName",
      "type": "server",
      "parameters": {
        "date": "$date",
        "time": "$time",
        "guests": "$number",
        "food_type": "$cuisine"
      },
      "credentials": "$private.my_credentials",
      "result_variable": "context.function_returned"
    }
  ]
}
```
NB. Be sure to replace `Namespace`, `PackageName`, `ActionName` by your values.
Here, we're changing the text sent back to the users and then telling the service where is the serverless action located and calling it with the values set by the user as parameters.
Then we're saving the return from the function in the `$function_returned` context variable
To check whether the call went through let's try to display the returned text.

5. Add a child node to the book_reservation node and set its condition to true 
![bottwo](https://github.com/IBMCodeLondon/chatbot-serverless-workshop/blob/master/bot2.gif?raw=true)
6. Edit the response using the JSON editor and add the following code that will retrieve the result from the function and display it:
```json
{
  "output": {
    "text": {
      "values": [
        "The function returned: $function_returned."
      ]
    }
  }
}
```
7. Open the book_reservation node and change the "And finally" condition to "Skip user input" so it automatically goes to the child node
![botthree](https://github.com/IBMCodeLondon/chatbot-serverless-workshop/blob/master/bot3.gif?raw=true)
8. Try it out! 

Great! You've just mixed two worlds together, how does it feel? 
Good and... still not complete! But you're only one step away.

## Lab 4: Call an external API from your serverless function (and do some color magic)

You've now connected your Assistant and your Cloud Function service. The latter will be used as a gateway to all the external APIs you can call such as, in our case, TripAdvisor, Booking.com... 
Obviously we don't want to spam restaurants with fake reservations, therefore we've put together a very simple API endpoint that expects the following:
```json
{
  "food_type":"",
  "date":"",
  "time":"",
  "guests":"",
  "hex_color":""
}
```
The four first parameters are from what you've gathered during the conversation, the fifth one will allow you to know whether the booking went through or not.

1. Return to the serverless action you have previously created
2. If you're using Node.js replace it with the following to do a POST on the API endpoint we've created for you:
```javascript
/** main() will be run when you invoke this action
  * @param Cloud Functions actions accept a single parameter, which must be a JSON object.
  * @return The output of this action, which must be a JSON object.
  */

  let rp = require('request-promise');
  
  function main(params) {
      // set the color you want the light to turn (in hexa)
      params.hex_color = "#00ff00";
      
      // set the API call and params
      let options = {
          method: 'POST',
          uri: 'https://icl-booking.eu-gb.mybluemix.net/book',
          body: {
                payload: params
            },
          headers: {
              'User-Agent': 'Request-Promise'
          },
          json: true
      };
       
      // call the API
      const promise = rp(options)
          .then(function (res) {
            return { message: res };   
          })
          .catch(function (err) {
              return { message: 'Sorry, something went wrong' };
          });
          
    return promise;
  }
```
3. Save and try to invoke it from the Functions page, it should fail and say that it's missing parameters
4. Try an other time from the Try it out panel on the Watson Assistant service

Did the light turn your color? If yes, celebrate! If no, let us know and we'll help you achieve it. 

## Lab 5 (bonus challenge!): Digressions

