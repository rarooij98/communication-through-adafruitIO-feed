# :love_letter: Communication through AdafruitIO feed
We'll be trying to communicate through a shared Adafruit IO feed using a button and an esp8266 NodeMCU board.
One person will press the button, and then the other person should be able to see the inputs on their screen.
This way we'll be able to say hi! :wave:

:calendar: **Date:** 12/10/2022

:writing_hand: **By:** Robin van Rooij & Cynthia de Vries

### What do you need for this?
- Adafruit IO account
- esp8266 NodeMCU board
- A button
- Wifi (not 5G)

## :computer: 1 - New feed 
Log into your Adafruit IO account and create a new feed. We named our feed 'kussiekussielove'.

<img src="https://github.com/rarooij98/communication-through-adafruitIO-feed/blob/main/images/newfeed.PNG" width=75% height=75%> 

<img src="https://github.com/rarooij98/communication-through-adafruitIO-feed/blob/main/images/love.PNG" width=40% height=40%>

To create a shared feed, open your feed and click Sharing. Invite the other person to your feed with their email and don't forget to give them READ + WRITE rights.

<img src="https://github.com/rarooij98/communication-through-adafruitIO-feed/blob/main/images/sharing.PNG" width=60% height=60%>

## :bellhop_bell: 2 - Button 
Connect the button to your board. We connected our button to D0 (red = 3.3, black to gnd, yellow to D0).

## :crystal_ball: 3 - Code 
Open your Arduino IDE and open an example sketch from Examples > adafruit io > Adafruitio_06_digital.in
Change the BUTTON_PIN to the one you're using, in our case we changed it to BUTTON_PIN D0.
Also put in your adafruit key and wifi password, you can do this in the config.h tab. Now your code is ready to go! :tada:

```
#define BUTTON_PIN D0

// button state
bool current = false;
bool last = false;

// set up the 'digital' feed
AdafruitIO_Feed *digital = io.feed("digital");

void setup() {
  // set button pin as an input
  pinMode(BUTTON_PIN, INPUT);

  // start the serial connection
  Serial.begin(115200);

  // wait for serial monitor to open
  while(! Serial);

  // connect to io.adafruit.com
  Serial.print("Connecting to Adafruit IO");
  io.connect();

  // wait for a connection
  while(io.status() < AIO_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  // we are connected
  Serial.println();
  Serial.println(io.statusText());
}

void loop() {
  // io.run(); is required for all sketches.
  // it should always be present at the top of your loop function. 
  // it keeps the client connected to io.adafruit.com, and processes any incoming data.
  io.run();

  // grab the current state of the button.
  // we have to flip the logic because we are using a pullup resistor.
  if(digitalRead(BUTTON_PIN) == LOW)
    current = true;
  else
    current = false;

  // return if the value hasn't changed
  if(current == last)
    return;

  // save the current state to the 'digital' feed on adafruit io
  Serial.print("sending button -> ");
  Serial.println(current);
  digital->save(current);

  // store last button state
  last = current;
}
```

## :envelope_with_arrow: 4 - Upload & check 
Upload your code. Check your serial monitor to see if the button is working properly.
It should say: "sending button ->", when the button input is being sent to your shared feed.

Then go to your Adafruit IO feed and check if you get a signal from the button.

### :triangular_flag_on_post: Error
Nothing happened when we pressed the button! We could see the confirmation message in the serial monitor, so we thought there must be a problem connecting to adafruit. We checked our code and found out we forgot to put in our feed name. Ofcourse it cannot connect to a feed it doesn't know.
We had to refer to our feed like this:

```
AdafruitIO_Feed *digital = io.feed("kussiekussielove");
```

### It works! :tada:
Now you can see the button input on your shared feed. It should look something like this:

<img src="https://github.com/rarooij98/communication-through-adafruitIO-feed/blob/main/images/output.PNG" width=75% height=75%>
<img src="https://github.com/rarooij98/communication-through-adafruitIO-feed/blob/main/images/log.PNG" width=50% height=50%>

## :card_file_box: Sources:
- https://learn.adafruit.com/adafruit-io-basics-feeds/sharing-a-feed
