---
published: true
title: Working with SMS via SMPP in NodeJS
---

Her's a short post aboun how to send and receive SMS messages from/to [SMSC](https://en.wikipedia.org/wiki/Short_message_service_center) with [SMPP](https://en.wikipedia.org/wiki/Short_Message_Peer-to-Peer) protocol in NodeJS.

For this, we will use the [node-smpp](https://github.com/farhadi/node-smpp) module (`npm install --save smpp`)

For beginning, you need to create a new sesion with IP address & port.

```js
var smpp = require('smpp');
var session = new smpp.Session({host: '0.0.0.0', port: 9500});
```

It will automatically connect, so we need to bind to the SMSC. In our case, we will bind as `transceiver`. We will do this on `connect` event: 

```js
// We will track connection state for re-connecting
var didConnect = false; 

session.on('connect', function(){
  didConnect = true;

  session.bind_transceiver({
      system_id: 'USER_NAME',
      password: 'USER_PASSWORD',
      interface_version: 1,
      system_type: '380666000600',
      address_range: '+380666000600',
      addr_ton: 1,
      addr_npi: 1,
  }, function(pdu) {
    console.log('pdu status', lookupPDUStatusKey(pdu.command_status));
    if (pdu.command_status == 0) {
        console.log('Successfully bound')
    }
  });
})
```

Here we are using a small helper `lookupPDUStatusKey` for easier debugging: 

```js
function lookupPDUStatusKey(pduCommandStatus) {
  for (var k in smpp.errors) {
    if (smpp.errors[k] == pduCommandStatus) {
      return k
    }
  }
}
```

We also want to automatically re-connect to the SMSC:

```js
function connectSMPP() {
  console.log('smpp reconnecting');
  session.connect();
}

session.on('close', function(){
  console.log('smpp disconnected')
  if (didConnect) {
    connectSMPP();
  }
})

session.on('error', function(error){
  console.log('smpp error', error)
  didConnect = false;
})
```

Great! we are connected! Now let's seee how to send out SMS:

```js
function sendSMS(from, to, text) {
  // in this example, from & to are integers
  // We need to convert them to String
  // and add `+` before
  
  from = '+' + from.toString();
  to   = '+' + to.toString();
  
  session.submit_sm({
      source_addr:      from,
      destination_addr: to,
      short_message:    text
  }, function(pdu) {
    console.log('sms pdu status', lookupPDUStatusKey(pdu.command_status));
      if (pdu.command_status == 0) {
          // Message successfully sent
          console.log(pdu.message_id);
      }
  });
}
```

Receiving incoming SMS is a bit more complicated - we need to attach to event `pdu` and listen to specific command `deliver_sm`. After receiving the SMS, we need to send back to SMSC information that we received the SMS, othervise SMSC will try to send us the same SMS again.

```js
session.on('pdu', function(pdu){

  // incoming SMS from SMSC
  if (pdu.command == 'deliver_sm') {
    
    // no '+' here
    var fromNumber = pdu.source_addr.toString();
    var toNumber = pdu.destination_addr.toString();
    
    var text = '';
    if (pdu.short_message && pdu.short_message.message) {
      text = pdu.short_message.message;
    }
    
    console.log('SMS ' + from + ' -> ' + to + ': ' + text);
  
    // Reply to SMSC that we received and processed the SMS
    session.deliver_sm_resp({ sequence_number: pdu.sequence_number });
  }
})
```

That's it! Feel free to ask questions.