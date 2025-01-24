# IBM-Watson-a-Facebook-Messenger-Chatbot-Powered-by-Watson-Conversation-using-ngrok
Creating a Facebook Messenger Chatbot powered by IBM Watson Conversation involves several steps, including setting up Watson Conversation (or the newer Watson Assistant), integrating with Facebook Messenger, and using a tool like ngrok for local development.

Below is a step-by-step guide along with the required code to set up your Facebook Messenger chatbot that is powered by IBM Watson Assistant. This example assumes you already have a basic understanding of Facebook's developer tools, ngrok, and Node.js.
Step-by-Step Guide
Prerequisites:

    IBM Watson Assistant: Create an instance of Watson Assistant in IBM Cloud.
    Facebook Developer Account: Set up a Facebook app and a Messenger bot.
    Node.js: Make sure Node.js is installed on your machine.
    ngrok: For exposing your local server to the internet (necessary for Facebook webhook).
    Facebook Page: You need a Facebook page to connect your Messenger bot to.
    ngrok: Download and install ngrok to tunnel local servers to the web.

1. Set Up IBM Watson Assistant

    Go to IBM Cloud and create a Watson Assistant service instance.
    Create a new Assistant and define your dialog (intents, entities, etc.) in the Assistant's workspace.
    Get your Watson Assistant credentials (API key and URL) from the Credentials section of your Watson Assistant service.

2. Set Up Facebook Developer Account

    Go to Facebook for Developers and create a new App.
    Under Messenger, create a Facebook Messenger Bot for the app.
    Create a Facebook Page to link your bot to. You'll need a Page Access Token for the bot to send messages.
    Set up your Webhook in the Facebook Developer Console with ngrok (or your public URL).

3. Set Up Your Local Server Using Node.js and Express

You’ll be using a Node.js server with the Express framework to interact with Facebook Messenger and Watson Assistant.

Install required packages:

npm init -y
npm install express body-parser request ibm-watson ngrok

4. Code the Node.js Server to Handle Webhooks and Communication

Here’s a simple server using Express to handle Facebook Messenger webhook events and integrate with Watson Assistant.

Create a file called app.js.

const express = require('express');
const bodyParser = require('body-parser');
const request = require('request');
const ngrok = require('ngrok');
const { IamAuthenticator } = require('ibm-watson/auth');
const AssistantV2 = require('ibm-watson/assistant/v2');

// Watson Assistant credentials
const assistant = new AssistantV2({
  version: '2021-11-27',
  authenticator: new IamAuthenticator({
    apikey: 'YOUR_WATSON_API_KEY',  // Replace with your Watson Assistant API key
  }),
  serviceUrl: 'YOUR_WATSON_URL',  // Replace with your Watson Assistant service URL
});

// Set up Facebook Messenger credentials
const PAGE_ACCESS_TOKEN = 'YOUR_FACEBOOK_PAGE_ACCESS_TOKEN';  // Replace with your page access token
const VERIFY_TOKEN = 'YOUR_VERIFICATION_TOKEN';  // Replace with your verification token

const app = express();

// Middleware to parse incoming requests
app.use(bodyParser.json());

// Facebook webhook setup
app.get('/webhook', (req, res) => {
  // Verify the webhook
  if (req.query['hub.mode'] === 'subscribe' && req.query['hub.verify_token'] === VERIFY_TOKEN) {
    res.status(200).send(req.query['hub.challenge']);
  } else {
    res.sendStatus(403); // Forbidden if token doesn't match
  }
});

// Handle messages from Facebook Messenger
app.post('/webhook', (req, res) => {
  const messagingEvents = req.body.entry[0].messaging;
  
  messagingEvents.forEach((event) => {
    const senderId = event.sender.id;
    
    if (event.message && event.message.text) {
      const text = event.message.text;

      // Send text to Watson Assistant and get response
      getWatsonResponse(text, senderId);
    }
  });

  res.sendStatus(200);
});

// Function to interact with Watson Assistant
async function getWatsonResponse(userInput, senderId) {
  try {
    const response = await assistant.message({
      assistantId: 'YOUR_ASSISTANT_ID',  // Replace with your Watson Assistant ID
      sessionId: 'YOUR_SESSION_ID',  // You can generate or use a static session ID
      input: {
        'message_type': 'text',
        'text': userInput
      }
    });

    const watsonResponse = response.result.output.generic[0].text;

    // Send Watson response back to Facebook Messenger
    sendTextMessage(senderId, watsonResponse);
  } catch (err) {
    console.log('Error from Watson Assistant: ', err);
  }
}

// Function to send text messages via Facebook Messenger API
function sendTextMessage(senderId, text) {
  request({
    url: 'https://graph.facebook.com/v11.0/me/messages',
    qs: { access_token: PAGE_ACCESS_TOKEN },
    method: 'POST',
    json: {
      recipient: { id: senderId },
      message: { text: text },
    },
  }, function (error, response, body) {
    if (error) {
      console.log('Error sending message:', error);
    } else if (response.body.error) {
      console.log('Error response:', response.body.error);
    }
  });
}

// Start the Express server
const server = app.listen(5000, () => {
  console.log('Server is listening on port 5000');
});

// Expose local server via ngrok (for Facebook webhook)
ngrok.connect(5000).then((url) => {
  console.log('ngrok URL: ', url);
});

5. Run the Server Locally

After setting up your code, run the server using the command below:

node app.js

ngrok will expose your local server to the internet with a public URL, which you can use for the Facebook Messenger webhook.
6. Set Up Webhook in Facebook Developer Console

    Go to your Facebook Developer Console > Messenger settings.
    Set your Webhook URL to the ngrok URL you get after running the server. This URL should look like https://<your-ngrok-id>.ngrok.io/webhook.
    Use the Verification Token from your code to complete the setup.

7. Test Your Messenger Bot

Once everything is set up, test the bot by sending a message on Facebook Messenger to your page. Your Facebook Messenger bot will forward the message to Watson Assistant, and Watson will respond accordingly.
Summary of Important Parts:

    ngrok is used to expose your local server to the internet for the Facebook Messenger webhook.
    Express is used to set up the webhook and handle messages.
    Watson Assistant (or Watson Conversation) is used to process the user’s input and provide responses.
    Facebook Messenger API is used to send and receive messages.

Important Notes:

    Make sure to handle the session ID correctly. In the code, I've used a placeholder for the session ID ('YOUR_SESSION_ID'), but ideally, you should create a unique session ID for each user or reuse the same session for ongoing conversations.
    Always secure your tokens and credentials, especially when deploying the application to production.

This setup provides a simple integration for a Facebook Messenger chatbot that uses IBM Watson Assistant to handle conversations. You can extend this functionality by adding more complex intents and entities to your Watson Assistant, and enhancing the message processing logic to accommodate more sophisticated use cases.
