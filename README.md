# Stop Watch

This project implements a basic stopwatch using an Arduino and a 7-segment display controlled by a shift register. The stopwatch can be paused and reset using physical buttons.

## Components Used

- Arduino Uno
- 74HC595 Shift Register (for controlling the 7-segment display)
- 4 x 7-Segment Displays
- Push Buttons (for pause and reset)
- Resistors
- Jumper Wires
- Breadboard

## Pin Configuration

- **CLOCK_PIN**: connected to the shift register's clock pin (Pin 9)
- **LATCH_PIN**: connected to the shift register's latch pin (Pin 8)
- **DATA_PIN**: connected to the shift register's data pin (Pin 7)
- **GND_DIGIT_PINS[]**: connected to the digit control pins for the 7-segment displays (Pins 2, 3, 4, 5)
- **BUTTONS[]**: connected to the buttons for pause and reset (Pins 12, 11)

## How It Works

- **Pause/Resume**: Press the pause button to pause the stopwatch. Press it again to resume.
- **Reset**: Press the reset button to reset the stopwatch to zero.
- The elapsed time is displayed on four 7-segment displays.

## Code

```cpp
// Constants
const int CLOCK_PIN = 9;
const int LATCH_PIN = 8;
const int DATA_PIN = 7;

const int GND_DIGIT_PINS[] = { 2, 3, 4, 5 };

const int BTN_PAUSE = 0;
const int BTN_RESET = 1;
const int BUTTONS[] = { 12, 11 };

// Untuk representasi digit 0~9
const byte digits[] = {
    0b11111100,
    0b01100000,
    0b11011010,
    0b11110010,
    0b01100110,
    0b10110110,
    0b10111110,
    0b11100000,
    0b11111110,
    0b11110110
};

// Global Variables
long pause_store = 0;               // Menyimpan waktu saat pause dilakukan
long millis_store = 0;              // Meyimpan millis() saat reset dilakukan
bool btn_ready[] = { true, true };  // [0] -> Pause Button, [1] -> Reset Button
bool pause_state = false;           // True -> Sedang pause, False -> Stopwatch berjalan

void setup() {
    pinMode( CLOCK_PIN, OUTPUT );
    pinMode( LATCH_PIN, OUTPUT );
    pinMode( DATA_PIN, OUTPUT );

    for( int i = 0; i < 4; i++ ) {
        pinMode( GND_DIGIT_PINS[i], OUTPUT );
        digitalWrite( GND_DIGIT_PINS[i], HIGH ); // default HIGH agar LED mati
    }

    pinMode( BUTTONS[BTN_PAUSE], INPUT_PULLUP );
    pinMode( BUTTONS[BTN_RESET], INPUT_PULLUP );
}

void loop() {
    check_button_state( BTN_PAUSE );
    check_button_state( BTN_RESET );
    update_display();
}

void check_button_state( int button ) {

    if( digitalRead(BUTTONS[button]) == LOW && btn_ready[button] ) {

        switch( button ) {
            case BTN_PAUSE:
            if( pause_state ) {
                millis_store = millis() - pause_store; // Melanjutkan Stop Watch
                pause_state = false;
            } else {
                pause_store = millis() - millis_store; // Pause Stop Watch
                pause_state = true;
            }
            break;

            case BTN_RESET:
                reset();
                break;
        }

        btn_ready[button] = false;

    } else if( digitalRead(BUTTONS[button]) == HIGH && !btn_ready[button] ) {
        btn_ready[button] = true;
    }
}

long power( int base, int exponent ) {
    if( exponent <= 0 ) {
        return 1;
    } else {
        return base * power( base, exponent - 1 );
    }
}

void update_display() {

    int segment;

    for( int i = 0; i < 4; i++ ) {
        if( pause_state ) {
            segment = ( (pause_store % power(10,6-i)) / power(10,5-i) );
        } else {
            segment = ( ((millis()-millis_store) % power(10,6-i)) / power(10,5-i) );
        }

        update_digit( i, segment );
    }
}

void update_digit( int digit, int val ) {

    byte segment = digits[val];

    if( digit == 2 ) {
        bitSet( segment, 0 );
    }

    digitalWrite( LATCH_PIN, LOW );
    shiftOut( DATA_PIN, CLOCK_PIN, MSBFIRST, segment );
    digitalWrite( LATCH_PIN, HIGH );

    digitalWrite( GND_DIGIT_PINS[digit], LOW );
    delay( 2 );

    digitalWrite( GND_DIGIT_PINS[digit], HIGH );
}

void reset() {
    millis_store = millis();

    if( pause_state ) {
        pause_store = 0;
    }
}
```

## How to Use

1. Connect the 7-segment displays, shift register, and buttons according to the pin configuration.
2. Upload the code to your Arduino.
3. Use the buttons to pause/resume and reset the stopwatch.
4. The elapsed time will be displayed on the 7-segment displays.

## Features

- Pause and resume functionality.
- Reset to zero.
- Display of elapsed time on 7-segment displays.

## License

This project is open-source and can be modified or distributed freely.
