---
layout: post
title: "Using GDB on MSP430"
modified:
categories: blog
excerpt: "Debugging MSP430 launchpad using GDB"
tags: [debugging]
comments: true
share: true
image:
  feature: bug.jpg
date: 2016-09-07T22:33:01-04:00
---

# Setting up MSP430 toolchain on linux

- mspdebug
- msp-gcc


# Simple RS-232 program to start

{% highlight c linenos %}
#include  <msp430g2553.h> 

void main(void)
{
	WDTCTL = WDTPW + WDTHOLD; // Stop WDT
	 
	P1DIR |= (BIT0|BIT6); // Set the LEDs on P1.0, P1.1, P1.2 and P1.6 as outputs
	P1OUT = 0x01;
	 
	BCSCTL1 = CALBC1_1MHZ; // Set DCO
	DCOCTL = CALDCO_1MHZ;
	P1SEL = BIT1 + BIT2 ;  // P1.1 = RXD, P1.2=TXD
	P1SEL2 = BIT1 + BIT2 ; // P1.1 = RXD, P1.2=TXD
	UCA0CTL1 |= UCSSEL_2;  // SMCLK
	UCA0BR0 = 104;         // 1MHz 9600
	UCA0BR1 = 0;           // 1MHz 9600
	UCA0MCTL = UCBRS0;     // Modulation UCBRSx = 1
	UCA0CTL1 &= ~UCSWRST;  // **Initialize USCI state machine**
	IE2 |= UCA0RXIE;       // Enable USCI_A0 RX interrupt
	 
	__bis_SR_register(LPM0_bits + GIE); // Enter LPM0, interrupts enabled
	
	while(1){

		};
}
 
// Echo back RXed character, confirm TX buffer is ready first
#pragma vector=USCIAB0RX_VECTOR
__interrupt void USCI0RX_ISR(void)
{
	while (!(IFG2&UCA0TXIFG));  // USCI_A0 TX buffer ready?
	UCA0TXBUF = UCA0RXBUF; 		// Echo back received character
	 
	if (UCA0RXBUF == 'R') P1OUT |= BIT0; // Turn on P1.0 red LED when R is received
	else if (UCA0RXBUF == 'r') P1OUT &= ~BIT0; // Turn off P1.0 red LED when r is received
	
	if (UCA0RXBUF == 'G') P1OUT |= BIT6; // Turn on P1.6 green LED when G is received
	else if (UCA0RXBUF == 'g') P1OUT &= ~BIT6; // Turn off P1.6 green LED when g is received
}

{% endhighlight %}

## Setup minicom/putty



# Simple debugging with gdb on msp430

First compile your file to an Elf file be sure to add -g flag to CFLAGS to let gdb see all symbls

    make

Then connect the device in mspdebug

    mspdebug rf2500

and erase the device in mspdebug

    (mspdebug) erase

Then let mspdebug accept commands sent by gdb

    (mspdebug) gdb

Start up gdb in a different terminal window and load the program to gdb (assuming main.elf is your compiled program)

    msp430-gdb "target remote localhost:2000" main.elf

When the above command did not work select target with

    (gdb) target remote localhost:2000

Then command mspdebug to load the Elf file  from gdb by sending load command

    (gdb) load main.elf

Set a breakpoint at the begin

    (gdb) br main
    (gdb) br main.c:32  //Breakpoint at line 32

Let the program continue (3 equivalent options)

    (gdb) continue
    (gdb) <press enter> 
    (gdb) c

When breakpoint is reached you can see values

    (gdb) info var          //print the defined global variables
    (gdb) info locals       //print the defined local variables
    (gdb) print teller      // prints value of teller
    (gdb) display teller    // displays value after every breakpoint

Delete a breakpoint

    (gdb) delete 1   // delete breakpoint 1

Let the program continue

    (gdb) continue

# More advanced examples

Hier hoe je commands kan toevoegen aan een breakpoint:  gdb/Break-Commands.html 
Dit in conditionele breakpoints en regex kan hele krachtige dingen opleveren:
Bijvoorbeeld, break op alle functies entries in specifieke file (“my_file.c”), maar alleen als status != 0, print dan een waarde, knipper mijn ledje, en ga door, sla ondertussen alles op in “my_debug_session.log” met timestamps.

Zo heb je met een paar regels conditionele tracing toegevoegd aan alle functies in een file met daarnaast nog een actie.
Dit soort constructies kan je ook weer opslaan in command-files om te delen met collega’s, of om volgende keer met 1 commando dit allemaal in te stellen.

(gdb) set debug timestamp

(gdb) set logging on
(gdb) set logging file my_debug_session.log
(gdb) rbreak my_file.c:. if status != 0
(gdb) commands
(gdb) silent
(gdb) printf "Status is %d\n",status
(gdb) call flash_my_led()
(gdb) cont
(gdb) end


# Great features

> Check if these work on MSP430??

Een mooie set van GDB features om te bekijken:
- record and replay – vollledige program state opslaan en opnieuw afspelen, handig voor reprodutie etc..
- reverse execution – terugstappen in tijd
- tracepoints – specifiek tracing voor remote-targets om realtime process niet teveel te verstoren
- watchpoints – breaken wanneer bepaalde waarde veranderd, zien wie je global beïnvloed in welke volgorde.
- Pretty printing 
- Dump and Restore-Files – dumpen en terguzetten van stukken memory (of specifieke variabelen) naar files

Als je hierna nog iets wil lezen : ddd 

# Resources

[Datasheet](http://www.ti.com/lit/ds/symlink/msp430g2553.pdf)
[Family user guide](http://www.ti.com/lit/ug/slau144j/slau144j.pdf)
[Header file](http://www.ece.utep.edu/courses/web3376/Links_files/msp430g2553.h)


