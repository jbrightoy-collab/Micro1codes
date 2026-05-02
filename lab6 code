/ PIC16F1829 Configuration Bit Settings
#include <xc.h>
#include <stdio.h>
#include <stdlib.h>

// CONFIG1
#pragma config FOSC = INTOSC    // Internal Oscillator
#pragma config WDTE = OFF       // Watchdog Timer disabled
#pragma config PWRTE = OFF      // Power-up Timer disabled
#pragma config MCLRE = ON       // MCLR pin enabled
#pragma config CP = OFF         // Flash Program Memory Protection disabled
#pragma config CPD = OFF        // Data Memory Protection disabled
#pragma config BOREN = OFF      // Brown-out Reset disabled
#pragma config CLKOUTEN = OFF   // Clock Out disabled
#pragma config IESO = OFF       // Internal/External Switchover disabled
#pragma config FCMEN = OFF      // Fail-Safe Clock Monitor disabled

// CONFIG2
#pragma config WRT = OFF        // Flash Memory Self-Write Protection off
#pragma config PLLEN = OFF      // 4x PLL disabled
#pragma config STVREN = ON      // Stack Overflow/Underflow Reset Enable
#pragma config BORV = LO        // Brown-out Reset Voltage low
#pragma config LVP = ON         // Low-Voltage Programming enabled

/* ----- Serial Configuration ----- */
#define BAUD 9600
#define FOSC 4000000L
#define DIVIDER ((int)(FOSC/(16UL * BAUD) -1))  // should be 25 for 9600/4MHz
#define NINE_BITS 0
#define SPEED 0x4
#define RX_PIN TRISC5
#define TX_PIN TRISC4

/* ----- Clock Macro ----- */
#define _XTAL_FREQ 4000000.0
#define ScanNum 4        // Change to 1 for single pad testing, 4 for full setup

/* ----- Function Prototypes ----- */
void setup_comms(void);
void putch(char);
int getch(void);
char getche(void);
void timer_config(void);
void clockAndpin_config(void);
void usartConfig(void);
void Display(int);

/* ========================= MAIN ============================ */
int main(int argc, char** argv) {
    unsigned int Tcount, Threshold;
    int Touch[4], mode;

    /* ----- TouchPad Setup ----- */
    CPSCON0 = 0x8C;   // Enable CPS, high-range oscillator
    CPSCON1 = 0x03;   // Default channel CPS3
    Touch[0] = 0x03;  // RA4 pad CPS3 (U - Green)
    Touch[1] = 0x09;  // RC7 pad CPS9 (T - Blue)
    Touch[2] = 0x05;  // RC1 pad CPS5 (S - Red)
    Touch[3] = 0x04;  // RC0 pad CPS4 (A - White)

    /* ----- System Configuration ----- */
    clockAndpin_config();
    usartConfig();
    setup_comms();

    /* ----- Threshold Setting (Adjust experimentally) ----- */
    Threshold = 0x1000; // Replace with your calibrated threshold

    /* ----- Main Loop ----- */
    while (1) {
        mode = 4; // default: no touch detected

        for (int j = 0; j < ScanNum; j++) {
            CPSCON1 = Touch[j];      // Select touch channel
            timer_config();
            while (!TMR0IF) continue; // Wait for Timer0 overflow

            // Read 16-bit timer value from Timer1
            Tcount = (TMR1H << 8) + TMR1L;
            TMR0IF = 0; // Clear Timer0 interrupt flag

            // Display measured value on serial monitor
            printf("Pad %d -> Tcount=%X  Threshold=%X\r\n", j, Tcount, Threshold);

            // If below threshold, a touch is detected
            if (Tcount < Threshold) mode = j;
        }

        // Show result on LEDs
        Display(mode);
    }
    return (EXIT_SUCCESS);
}

/* ===================== DISPLAY FUNCTION ===================== */
void Display(int delay) {
    switch (delay) {
        case 0: // U - Green LED
            LATA &= ~(1 << 5);
            __delay_ms(500);
            LATA |= (1 << 5);
            __delay_ms(500);
            break;

        case 1: // T - Blue LED
            LATA &= ~(1 << 2);
            __delay_ms(500);
            LATA |= (1 << 2);
            __delay_ms(500);
            break;

        case 2: // S - Red LED
            LATC &= ~(1 << 6);
            __delay_ms(500);
            LATC |= (1 << 6);
            __delay_ms(500);
            break;

        case 3: // A - White (All LEDs Blink)
            LATA &= ~((1 << 5) | (1 << 2));
            LATC &= ~(1 << 6);
            __delay_ms(500);
            LATA |= (1 << 5) | (1 << 2);
            LATC |= (1 << 6);
            __delay_ms(500);
            break;

        case 4: // No touch
        default:
            LATA |= (1 << 5) | (1 << 2);
            LATC |= (1 << 6);
            break;
    }
}

/* =================== SYSTEM CONFIGURATION =================== */
void clockAndpin_config() {
    OSCCON = 0x6A;   // 4 MHz internal clock
    INTCON = 0x00;   // Disable interrupts
    OPTION_REG = 0xC5; // Prescaler 1:64, Fosc/4 input for Timer0
    T1CON = 0xC1;    // Enable Timer1
    T1GCON = 0x81;   // Timer1 Gate Enable (no toggle)
    TRISA = 0x10;    // RA5 (Green), RA2 (Blue) outputs; RA4 input
    TRISC = 0xBF;    // RC6 (Red LED) output, others input
    PORTA = 0x00;
    ANSELA = 0x10;   // RA4 analog
    ANSELB = 0x70;   // PortB analog inputs (RB7..RB4)
    ANSELC = 0x00;   // PortC all digital (UART RC4 & RC5)
}

/* ===================== TIMER CONFIGURATION ================== */
void timer_config() {
    TMR1ON = 0;
    TMR0 = 0;
    TMR1H = 0;
    TMR1L = 0;
    TMR1ON = 1;
    TMR0IF = 0;
    TMR0 = 0;
}

/* ===================== USART CONFIGURATION ================== */
void usartConfig() {
    APFCON0 = 0x84;  // Set RC5/RC4 as RX/TX
    TXCKSEL = 1;
    RXDTSEL = 1;
}

void setup_comms(void) {
    RX_PIN = 1;
    TX_PIN = 1;
    SPBRG = DIVIDER;
    RCSTA = (NINE_BITS | 0x90);
    TXSTA = (SPEED | NINE_BITS | 0x20);
    TXEN = 1;
    SYNC = 0;
    SPEN = 1;
    BRGH = 1;
}

/* ===================== SERIAL FUNCTIONS ===================== */
void putch(char byte) {
    while (!TXIF) continue; // Wait until TXREG empty
    TXREG = byte;
}

int getch(void) {
    while (!RCIF) continue; // Wait until RXREG full
    return RCREG;
}

char getche(void) {
    char c;
    putch(c = getch());
    return c;
}
