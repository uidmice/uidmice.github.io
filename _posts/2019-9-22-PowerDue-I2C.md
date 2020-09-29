---
layout: post
title: I2C Communication Between PowerDués
tag: [Hardware]
---
### I²C Communication between PowerDués

I²C, pronounced as "I squared C”, is an abbreviation for “Inter-Integrated Circuit”. It is a serial protocol used on a low-speed 2-wire interface. The two signal lines are:
  * **SDA** - the bidirectional data line
  * **SCL** - the clock signal
along with a power and a ground connection.

![_config.yml]({{ site.baseurl }}/images/i2c-interface.png)

To interface the I2C bus, devices are divided into Masters and Slaves.
  * Master device - controls the bus and supplies the clock signal. It requests data from the slaves individually. They do not have an address assigned to them.
  * Slave device - has a 7-bit address, and the address is unique.

### Device Setup

The PowerDué has SDA and SCL connections on the external breadboard that can be used for I²C communications between devices. The core Arduino Dué has two I²C interfaces: SDA(pin20), SCL(pin21) and extra SDA1 and SCL1. SDA and SCL have internal pullup resistors, but SDA1 and SCL1 do not. The SDA and SCL connections of the PowerDué uses the extra SDA1 and SCL1 of the core.

![_config.yml]({{ site.baseurl }}/images/i2cpins.png)

To set up, connect together the SDA, SCL and the ground of the two PowerDués. Remember to add two 1k5Ω pullup resistors to each of the SDA and SCL lines.

![_config.yml]({{ site.baseurl }}/images/i2c_setup.png)
  * Green wire - SDA
  * Blue wire - SCL
  * Black wire - ground
  * Red wire - 3.3V
  * Red boxes circle a total of 1k5Ω as pullup resistors


### Wire library

We use the built-in Wire library to control the I²C communications. The library has the following API for PowerDué:

  * **Wire1.begin(SLAVE_ADDR)** - Init the I2C communication with 7-bit slave address. If SLAVE_ADDR not specified, the device joins the bus as a master, otherwise a slave.
  * **Wire1.available()** - Returns the number of bytes available for retrieval with read().
  * **Wire1.read()** - Reads a byte that was transmitted from a slave device to a master or from a master to a slave.
  * **Wire1.write(DATA, LENGTH | VALUE | STRING)** - Writes data from a slave device in response to a request from a master, or queues data to be sent from a master to a slave.

For masters:
  * **Wire1.beginTransmission(SLAVE_ADDR)** - Begin a transmission to the I2C slave device with the given address. Subsequently, queue bytes for transmission with the write() function and transmit them by calling endTransmission().
  * **Wire1.endTransmission()** - Ends a transmission to a slave device that was begun by beginTransmission() and transmits the bytes that were queued by write().
  * **Wire1.requestFrom(SLAVE_ADDR,ANSWER_SIZE)** -  Request bytes from a slave device.

For slaves:
  * **Wire1.onRequest(FUNC_HANDLER)** - Registers a function to be called when a slave device receives a request from a master.The function should take no parameter and return nothing. <code C>void FUNC_HANDLER()</code>
  * **Wire1.OnReceive(FUNC_HANDLER)** - Registers a function to be called when a slave device receives a transmission from a master.The function should take a single int parameter (the number of bytes read from the master) and return nothing. <code C>void FUNC_HANDLER(int NUM_BYTES)</code>


(For those that only have one I2C interface, use Wire.func() instead of Wire1.func() for the above functions. For example, Uno, Mega2560, Ethernet.)

(For Arduino Dué, Wire.func() controls the I2C interface on pin 20 and 21, which is not accessible on our PowerDué. Therefore, use Wire1.func() to control the extra interface SDA1 and SCL1 available on PowerDué. )

### Code

**Slave**

<pre><code>
#include &lt;Wire.h&gt;

#define SLAVE_ADDR 9
#define ANSWER_SIZE 5

String answer = "Halo!";

void setup() {

  Wire1.begin(SLAVE_ADDR); // Initialize I2C communications as Slave
  Wire1.onRequest(requestEvent);
  Wire1.onReceive(receiveEvent);

  SerialUSB.begin(9600); // Setup Serial Monitor
  while(!SerialUSB); //Wait for the serial to set up
  SerialUSB.println("I2C Slave Demonstration");
}

void receiveEvent(int b) {
  byte x;
  // Read while data received
  while (0 < Wire1.available()) {
    x = Wire1.read();
  }

  // Print to Serial Monitor
  SerialUSB.print("Slave - receive ");
  SerialUSB.print(x);
  SerialUSB.println(" from the master.");


}

void requestEvent() {
  byte response[ANSWER_SIZE];

  // Format answer as array
  for (byte i=0;i<ANSWER_SIZE;i++) {
    response[i] = (byte)answer.charAt(i);
  }

  // Send response back to Master
  Wire1.write(response,sizeof(response));
  SerialUSB.print("Slave - send ");
  SerialUSB.print(answer);
  SerialUSB.println(" to the master.");}

void loop() {
   delay(1000);
}
</code></pre>


**Master**

<pre><code>
#include &lt;Wire.h&gt;

#define SLAVE_ADDR 9
#define ANSWER_SIZE 5

void setup() {

  Wire1.begin();  // Initialize I2C communications as Master

  SerialUSB.begin(9600);   // Setup serial monitor
  while(!SerialUSB); //Wait for the serial to set up
  SerialUSB.println("I2C Master Demonstration");
}

void loop() {
  delay(1000);
  SerialUSB.println("Master - write data to slave");

  // Write a charatre to the Slave
  Wire1.beginTransmission(SLAVE_ADDR);
  Wire1.write(0);
  Wire1.endTransmission();
  delay(500);

  Wire1.requestFrom(SLAVE_ADDR,ANSWER_SIZE);

  // Add characters to string
  String response = "";
  while (Wire1.available()) {
      char b = Wire1.read();
      response += b;
  }

  // Print to Serial Monitor
  SerialUSB.print("Master - receive ");
  SerialUSB.print(response);
  SerialUSB.println(" from the slave.");
}</code></pre>

### Demo
<video width="320px" controls>
  <source src="{{ site.baseurl }}/images/i2c_demo.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
