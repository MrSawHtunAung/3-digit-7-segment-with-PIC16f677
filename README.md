# 3-digit-7-segment-with-PIC16f677 with CCS(PCWHD) compiler 
Here is the sample code for interfacing 3 digits 7 segments display. 

Connect C0 to C6 to the segment A to G.

Connect A0 to A2 to the digit 1 to 3.


## Definitions and Declarations:
```cpp
#include <16F677.h>                 // Include the PIC16F677 header file
#fuses INTRC_IO,NOWDT,PUT,NOPROTECT,BROWNOUT,NOMCLR  // Configuration fuse settings
#use delay (clock=8000000)          // Set the clock frequency for the delay function

#byte TRISA = 0b10000101 	   // Define TRISA register address and initialize it
#byte PORTA = 0b00000101 	   // Define PORTA register address and initialize it

#byte TRISC = 0b10000111 	   // Define TRISC register address and initialize it
#byte PORTC = 0b00000111 	   // Define PORTC register address and initialize it

#define SPORTA PORTA     	   // Define SPORTA as an alias for PORTA
#define SPORTC PORTC               // Define SPORTC as an alias for PORTC

const char SegCode[10] = {0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D, 0x07, 0x7F, 0x6F};  	    // for Common Cathode 7-segment display codes
//const char SegCode[11] = {0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8, 0x80, 0x90};    	  // for Common Anode 7-segment display codes
const char Column[3] = {0x01,0x02,0x04};  	// Define column values for the display
static char Segment[3] = {0x7f,0x7f,0x7f};  	// Initialize segment array with off values
static unsigned char ColCount = 0x00;  		// Initialize column count variable
```
## Display Function:
```cpp
void Display() {
   PORTA = 0b00100111; 			// Turn off all digits
   PORTC = 0b00111111; 			// Turn off segments a-f
   delay_cycles(1);  			  // Delay 
   if (ColCount >= 3) ColCount = 0;  	  // Reset column count if it exceeds 3
   SPORTC = Segment[ColCount];  	      // Set PORTC to the current segment
   PORTA = Column[ColCount]^0x07;       // Set PORTA based on the current column
   ColCount++;  			                  // Increment column count
}
```
## Timer ISR: 200us timer
```cpp
#INT_TIMER0
void Timer0(void) { 
   Display(); 					// Display function will display every 200us
}
```
## Config Function: configuration for Ports and timer 
```cpp
void CONFIG() {
   TRISA= 0b00011000; PORTA=0b00100111;  	// Configure and initialize TRISA and PORTA
   TRISC= 0b00000000; PORTC=0b00110111;  	// Configure and initialize TRISC and PORTC

   setup_timer_0(T0_INTERNAL | T0_DIV_8);  	// Configure Timer0 with internal clock and 1:8 prescaler *you can change DIV_8 to 16 for 400us
   set_timer0(56);  				         // Set Timer0 initial value for a 200us interrupt at 8MHz

   enable_interrupts(GLOBAL);  			 // Enable global interrupts
   enable_interrupts(INT_TIMER0);  	 // Enable Timer0 interrupt
}
```
## Show Function: to show number to each segments
```cpp
void Show(unsigned int32 Num) { 
   Segment[0] = SegCode[(Num / 100)];  		  // Set the first segment based on hundreds digit
   Segment[1] = SegCode[(Num / 10) % 10];  	// Set the second segment based on tens digit
   Segment[2] = SegCode[ Num % 10 ];  		  // Set the third segment based on units digit
}
```
## Main Loop: 
```cpp
void main() {
   CPU_SETUP(); 
   while (true){
      Show(123);  		// The number that you want to display.
   }
}  
```

THANKS YOU.
