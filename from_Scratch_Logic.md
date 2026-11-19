Problem:
**How does MCU B Know that MCU A has finished transmitting a frame? & How do both MCUs know their respective responsibilities?**

MCU A is programmed to generate the Frame. MCU B is programmed to detect and decode it. AKA one way communication system.

<img width="300" height="90" alt="download" src="https://github.com/user-attachments/assets/03e0d577-1da1-43fa-8acc-dc61120aefa0" />
<svg font-family="-apple-system-body, ui-sans-serif, -apple-system, system-ui, &quot;Segoe UI&quot;, Helvetica, &quot;Apple Color Emoji&quot;, Arial, sans-serif, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;" font-weight="400" data-d-component="svg" fill="currentColor" style="color:rgb(237, 237, 237)" viewBox="0 0 320 96" width="100%" xmlns="http://www.w3.org/2000/svg"><rect x="5" y="12" width="110" height="68" rx="8" fill="#0D9488" fill-opacity=".1" stroke="#0D9488"/><rect x="205" y="12" width="110" height="68" rx="8" fill="#2563EB" fill-opacity=".1" stroke="#2563EB"/><text x="60" y="36" font-size="14" font-weight="600" text-anchor="middle" fill="currentColor"></text><text x="60" y="54" font-size="11" text-anchor="middle" fill="currentColor"></text><text x="60" y="68" font-size="10" text-anchor="middle" fill="currentColor"></text><text x="260" y="36" font-size="14" font-weight="600" text-anchor="middle" fill="currentColor"></text><text x="260" y="54" font-size="11" text-anchor="middle" fill="currentColor"></text><text x="260" y="68" font-size="10" text-anchor="middle" fill="currentColor"></text><path d="M115 45 H203 M195 39 L203 45 L195 51" fill="none" stroke="currentColor" stroke-width="2"/><text x="160" y="33" font-size="11" text-anchor="middle" fill="currentColor"></text></svg>

I provide **start bit** at the beginning of the data frame before MCU A transmits to the MCU B, we know that any hardware with communication protocol is required to be set up the DATA line either HIGH or LOW, physically.

For UART, the standard rule is that DATA LINE is **always set to HIGH** so we can always put **Start Bit** at the front of data frame.

for eg.: 
data_frame = 10110010, 
start_bit = 0
MCU B data_line = 1

MCU A will send start_bit + data_frame.  The start bit will set Data Line to LOW and MCU B will know it is receiving data frame. 

similarly, if: 
data_frame = 00111010
start_bit = 0
MCU B data_line = 1

MCU B will know that it is receiving data frame, but subsequently it finds another bit 0, now its not big problem here, cuz MCU B will sample it using **time-based bit sampling.**

Till now this theory founds the identification of reception at receiver side. 

The subsequent relevant question is MCU B knowing the **finish of a data frame.** An obvious answer to guess for the first time is add a delay after the data frame finished but the that's not it.
Adding delay won't solve anything but only increase the start of data frame 2. Suppose:

|Parameter|	Agreed value|
| ---| --- |
|Idle state|	HIGH|
|Start bit|	LOW|
|Data bits|	8|
|Bit duration|	1 ms|
|Stop bit|	HIGH|
|Additional idle gap|	5 ms|

Here's what happens inside MCU B:
- It detects the falling edge of the start bit and establishes the timing reference.
- It uses a timer and a counter to sample the eight data bits.
- After sampling the eighth bit, it expects a HIGH stop bit and checks it around 9.5 ms.
- If the stop bit is valid, it declares the frame complete and can deliver the received byte.
- It then waits for the next start bit, which resynchronizes the next frame.

Now, when MCU B checks the stop bit at 9.5 ms, the DATA line is still LOW instead of HIGH.  What should MCU B conclude? **Simply add a start bit checker at each data frame not data bit.**

