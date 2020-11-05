# BMILmao
// See https://github.com/dialogflow/dialogflow-fulfillment-nodejs
// for Dialogflow fulfillment library docs, samples, and to report issues
'use strict';
 
const functions = require('firebase-functions');
const {WebhookClient} = require('dialogflow-fulfillment');
const {Card, Suggestion, Payload} = require('dialogflow-fulfillment');

const admin = require('firebase-admin');
admin.initializeApp({
  credential: admin.credential.applicationDefault(),
  databaseURL: "https://justadumbbot-72bd1.firebaseio.com/"
});
 
process.env.DEBUG = 'dialogflow:debug'; // enables lib debugging statements
 
exports.dialogflowFirebaseFulfillment = functions.https.onRequest((request, response) => {
  const agent = new WebhookClient({ request, response });
  console.log('Dialogflow Request headers: ' + JSON.stringify(request.headers));
  console.log('Dialogflow Request body: ' + JSON.stringify(request.body));
 
  function welcome(agent) {
    agent.add(`Welcome to my agent!`);
  }
 
  function fallback(agent) {
    agent.add(`I didn't understand`);
    agent.add(`I'm sorry, can you try again?`);
  }
  
  function bodyMassIndex(agent) {
  	let weight = request.body.queryResult.parameters.weight;
    let height = request.body.queryResult.parameters.height / 100;
    let bmi = (weight / (height * height)).toFixed(2);
    
    let result = "none";
    let pkgId = "1";
    let stkId = "1";
    
    if (bmi < 18.5) {
    	   result = "xs";
        pkgId = "11538";
        stkId = "51626519";
    } else if (bmi >= 18.5 && bmi < 23) {
    	   result = "s";
        pkgId = "11537";
        stkId = "52002741";
    } else if (bmi >= 23 && bmi < 25) {
    	   result = "m";
      	 pkgId = "11537";
        stkId = "52002745";
    } else if (bmi >= 25 && bmi < 30) {
    	   result = "l";
      	 pkgId = "11537";
        stkId = "52002762";
    } else if (bmi >= 30) {
    	   result = "xl";
      	 pkgId = "11538";
        stkId = "51626513";
    }
    
    let payloadJson = {
    	  "type": "sticker",
      	"packageId": pkgId,
       "stickerId": stkId
    };
    
    let payload = new Payload("LINE", payloadJson, {sendAsMessage: true});
    
    //Realtime Database
    //return admin.database().ref("bmi").child(result).once('value').then(snapshot => {
    //	agent.add(snapshot.val());
    //});
    
    //Cloud Function
    return admin.firestore().collection('bmi').doc(result).get.then( doc => {
      agent.add(payload);	
      agent.add(doc.data().description);
    });
  }
  
  let intentMap = new Map();
  intentMap.set('Default Welcome Intent', welcome);
  intentMap.set('Default Fallback Intent', fallback);
  intentMap.set('BMI - custom - yes', bodyMassIndex);
  agent.handleRequest(intentMap);
});
