*“ChallengePost’s mission is to celebrate software and the people who make it.  This mission is firmly rooted in the belief that ordinary people can bring about creative technological solutions to big issues and challenges facing our world.”*

![arduino1]({{ site.baseurl }}/assets/hacking-the-office/arduino1.jpg)

Seeing that we are a hackathon platform, it is nearly impossible to spend a day at the ChallengePost offices without hearing about some exciting hack that a team member or one of our awesome community members has created.  While most of our team is located in New York City we also have a few awesome remote team members.  Yes, our remote team is spread across the country, but they are still just as involved in everyday CP activities as any other team member.  Between attending meetings via google hangouts and actively participating in hilarious conversations on our various Slack channels it is pretty easy to forget that they could be in a different time zone.

One team member that is not working remotely, however, is our developer evangelist - Neal Shyam.  Neal is a true hacker.  When he isn’t highlighting epic open source projects on his blog, he’s most likely adding to his ever-growing collection of chrome extensions and various other hacks.  When one of our amazing Project Manager, Serena, made the transition to remotely working form Boulder, CO, Neal, in true hacker fashion, wasted no time in providing a solution to quickly reaching Serena.  And yes, it did involve a chrome extension.  Click the Serena button in your browser and a Hangout instantly begins - epic.


![toolbar]({{ site.baseurl }}/assets/hacking-the-office/toolbar.png)

After a week or so of playing with the button I got to thinking about how cool it would be to have a physical call Serena button.  I had been messing around with my new Arduino for the past few weeks and after chatting with the team, we decided this would make an awesome project. This leads us to version 2.0 of the Call Serena chrome extension - the Call Serena Button.

(source code so you can follow along)

Bridging the gap between hardware and web development is something I’ve become increasingly excited about.  Working with web applications is fun, Bringing those web applications to the physical world is on a whole different level of exciting.  With an abundance of powerful microcontrollers priced below $40 and more open source APIs than seconds in a day, there really is no excuse to not start experimenting with these technologies today.  Our project required three things.

1. Arduino Uno

2. Chrome Serial API

3. Inspiration

The Arduino is a microcontroller board with 14 digital input/output pins, 6 analog inputs, USB connection, and power jack.  In other words it is a small computer capable of big things.

![arduino2]({{ site.baseurl }}/assets/hacking-the-office/arduino2.jpg)

There are two main ways to build on top of Chrome - Extensions and Packaged Apps.

Chrome describes extensions as “small programs that add new features to your browser and personalize your browsing experience.”  There are tens of thousands of these available to download and the possibilities are endless.  Extensions can range from beautiful productivity apps such as Momentum, to ridiculous apps from yours truly.

One of the newest additions to Chrome’s development platform, and something I’m extremely stoked on, are Packaged Apps.  These applications “have access to Chrome APIs and services not available to traditional web sites. You can build powerful apps that interact with network and hardware devices, media tools…”  The best part is that you build them in languages and tools familiar to web developers such as JavaScript, CSS, and HTML.  Seeing that we would be interfacing with a hardware device we decided to build a packaged app using Chrome’s Serial API - just one of the many awesome javascript APIs available to developers building on their platform

The best way to fully understand how everything ties together is to walk through the code the same way it is executed - starting from the Arduino in our NYC office and making our way to Serena in Boulder, CO.

At the top of the Arduino we have our USB cable, which also serves as the power source while connected.  On the left side of the board we have two wires.  A red wire is connected to 5v (providing the power) and a black wire is connected to GND, which stands for ground.

![arduino3]({{ site.baseurl }}/assets/hacking-the-office/arduino3.jpg)

On the breadboard you’ll notice we have a large red button, which when pressed completes the circuit.  Between our button and ground we have an orange wire, which is connected to digital input 7 on the Arduino.  We will read from this pin to know when the circuit has been completed.

![arduino1]({{ site.baseurl }}/assets/hacking-the-office/arduino4.jpg)

Every Arduino sketch needs a setup and loop function.  On line 1 and 2 we declare two variables.  We give led a value of 7 and val a value of -1. In our setup function, on line 5, we open a serial port and set the data rate to 9600 bits/second.  We then use an Arduino function pinMode to set pin 7 to INPUT, since we’re going to want to read its input.  Once an Arduino sketch is uploaded to the board it runs a continuous loop.  Every time the loop runs we read pin 7 to check to see if it has been set to 1 indicating HIGH.  If this is true we know the circuit has been completed, indicating the button was pressed, and therefore write two bytes (“HI”) to our serial connection and delay 5000 millis  ~ 5 seconds - a cheap way of debouncing.  This is it for the Arduino.  Now we can turn our attention to the Packaged App - where most of the magic happens.

```arduino
int led = 7;
int val = -1;

void setup() {
  Serial.begin(9600);
  pinMode(led, INPUT);
}

void loop() {
  int value = digitalRead(led);
  if (value == 1) {
    Serial.write("HI");
    delay(5000);
  }
}
```


Packaged Apps have a few files you should be familiar with.

1. Manifest.json.  This file is required and contains important information about our app including the name, description, scripts to run, and any required permissions or Chrome APIs.

    ```json
    {
      "name": "Call Serena Button",
      "description": "Call Serena from Arcade button",
      "version": "0.1",
      "manifest_version": 2,
      "app": {
      "background": {
          "scripts": ["background.js"]
        }
      },
      "permissions":["serial"],
      "icons": {
        "16": "button-16.png",
        "128": "button-128.png"
      }
    }
    ```

2. HTML file to provide the app’s user interface.

    *window.html*

    ```html
    <!DOCTYPE html>
    <html>
      <head>
      </head>
       <body>
         <div id='serial-port'></div>
         <div id='status'></div>
        <script src="script.js"></script>
      </body>
    </html>
    ```

3. background script is used to create the event page responsible for managing the app life cycle.

    background.js

    ```javascript
    chrome.app.runtime.onLaunched.addListener(function() {
      chrome.app.window.create('window.html', {
        'bounds': {
          'width': 400,
          'height': 500
        }
      });
    });
    ```

4. JavaScript file(s) to run

    *script.js*

    ```javascript
    //sets global variable to check if connection failed by not returning a connectionId
    var connectionId = -1

    function sendData(connectionId, bytes) {
      var buffer = new ArrayBuffer(bytes);
      chrome.serial.send(connectionId, buffer, function(sendInfo) {console.log(sendInfo) })
    }

    function setConnectionStatus(connectionId, successStatus) {
      var statusDiv = document.getElementById('status')

      if (connectionId == -1) {
        statusDiv.innerHTML = 'Connection failed!'
      }
      else if (connectionId > 0) {
        statusDiv.innerHTML = successStatus + ' Connection ID: ' + connectionId
      }
    }

    function onOpen(openData) {
      var connectionId = openData.connectionId;
      setConnectionStatus(connectionId, 'Succesfully connected');
      chrome.serial.onReceive.addListener(function(info) { window.open("<Serena's URL>") })
    }

    function openPort() {
      var connectedPort = document.getElementById('serial-port').innerHTML
      chrome.serial.connect(connectedPort, onOpen);
    }

    function getPortPath(ports) {
        var div = document.getElementById('serial-port');
        var portPath = ports[5].path
        div.innerHTML = portPath;
    }

    chrome.serial.getDevices(function(ports) {
      getPortPath(ports);
      openPort();
    });
    ```

When the app is launched background.js is executed, which opens window.html, thus loading and running script.js.

The chrome serial API provides many useful methods that we take advantage of to get our Arduino communicating with our Packaged App. Any method prefixed by chrome.serial is provided by the API.  `getDevices` executes first and takes a callback parameter we’ve named ports since it is an array of all available ports.  Next `getPortPaths` is called, passing in the ports array, setting the div with id ‘serial-port’ to the port path - dev/tty.usbmodemfa131.

![arduino1]({{ site.baseurl }}/assets/hacking-the-office/window.png)

Once this function is done executing `openPort` is called.  This function grabs the port path from the div and attaches it to a variable, `connectedPort`.  We then connect to this port with `connect`, which takes two arguments,

1. Path of the port you want to connect to

2. Success callback - `onOpen`, taking a callback parameter of openData which renders the connection status on the window.html page.

Then comes arguably the most important function of the file `addListener`.  This sets an event listener, which is very similar to web development, only instead of listening for user executed events we are listening for any data to get sent through the connected port.  On successfully receiving data it executes a function with a callback parameter of info.  Google hangouts work with unique url’s, similar to a phone number.  In our case all we need to do is open a new window with Serena’s hangout address.  Our application have done their job and Serena receives an incoming Hangout request.

One of the biggest take aways from this project is that just by including a simple switch developers can easily bridge the gap between the physical and digital worlds.  You don’t need to limit yourself to just opening a new window upon receiving data.  You could just as easily call upon one of the seemingly infinite number of APIs available today.  The switch just kicks off your application.  Once back on the internet, where us web developers are truly comfortable, the possibilities are truly limitless.

Have questions or comments?  Tweet us @askwheeler , @nealrs, or @challengepost

Source Code: [https://github.com/AlexWheeler/Serena-Button](https://github.com/AlexWheeler/Serena-Button)
