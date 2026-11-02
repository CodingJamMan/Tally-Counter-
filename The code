#include <TM1637Display.h>
#include <EEPROM.h>

// Hardware Pin Definitions
const uint8_t CLK = 3;
const uint8_t DIO = 2;
const uint8_t UP_BUTTON = 4;
const uint8_t RESET_BUTTON = 5;
const uint8_t DOWN_BUTTON = 6;

TM1637Display display(CLK, DIO);

int count = 0;
const int eepromAddress = 0;

// Independent Button States and Debounce Timers
bool currentUpState = HIGH;
bool lastUpState = HIGH;
unsigned long lastUpDebounceTime = 0;

bool currentDownState = HIGH;
bool lastDownState = HIGH;
unsigned long lastDownDebounceTime = 0;

bool currentResetState = HIGH;
bool lastResetState = HIGH;
unsigned long lastResetDebounceTime = 0;

const unsigned long DEBOUNCE_DELAY = 50;

// Timing Variables
unsigned long lastButtonPressTime = 0;
bool needsSaving = false;
const unsigned long EEPROM_SAVE_DELAY = 2000;

// Custom Segment Definitions for Snake Animation
#define SEG_A 0b00000001
#define SEG_B 0b00000010
#define SEG_C 0b00000100
#define SEG_D 0b00001000
#define SEG_E 0b00010000
#define SEG_F 0b00100000
#define SEG_G 0b01000000

void setup() {
  display.setBrightness(0x0f);

  // --- SNAKE SPLASH SEQUENCE ---
  // Array holding the explicitly sized 4-digit segment map frames
  uint8_t snakeFrames[][4] = {
    { SEG_A, 0, 0, 0 },
    { 0, SEG_A, 0, 0 },
    { 0, 0, SEG_A, 0 },
    { 0, 0, 0, SEG_A },
    { 0, 0, 0, SEG_B },
    { 0, 0, 0, SEG_G },
    { 0, 0, SEG_G, 0 },
    { 0, SEG_G, 0, 0 },
    { SEG_G, 0, 0, 0 },
    { SEG_E, 0, 0, 0 },
    { SEG_D, 0, 0, 0 },
    { 0, SEG_D, 0, 0 },
    { 0, 0, SEG_D, 0 },
    { 0, 0, 0, SEG_D },
    { 0, 0, 0, SEG_C },
    { 0, 0, 0, 0 } // Clear screen at the end
  };

  int totalFrames = sizeof(snakeFrames) / sizeof(snakeFrames[0]);
  for (int i = 0; i < totalFrames; i++) {
    display.setSegments(snakeFrames[i]);
    delay(100); // Animation speed (lower means faster snake)
  }

  // Load counter value after splash screen finishes
  EEPROM.get(eepromAddress, count);
  if (count < 0 || count > 9999) {
    count = 0;
  }

  pinMode(UP_BUTTON, INPUT_PULLUP);
  pinMode(DOWN_BUTTON, INPUT_PULLUP);
  pinMode(RESET_BUTTON, INPUT_PULLUP);

  updateDisplay();
}

void loop() {
  unsigned long currentMillis = millis();

  // --- UP BUTTON ---
  bool readingUp = digitalRead(UP_BUTTON);
  if (readingUp != lastUpState) {
    lastUpDebounceTime = currentMillis;
  }
  if ((currentMillis - lastUpDebounceTime) > DEBOUNCE_DELAY) {
    if (readingUp != currentUpState) {
      currentUpState = readingUp;
      if (currentUpState == LOW) {
        handleUpButton();
      }
    }
  }
  lastUpState = readingUp;

  // --- DOWN BUTTON ---
  bool readingDown = digitalRead(DOWN_BUTTON);
  if (readingDown != lastDownState) {
    lastDownDebounceTime = currentMillis;
  }
  if ((currentMillis - lastDownDebounceTime) > DEBOUNCE_DELAY) {
    if (readingDown != currentDownState) {
      currentDownState = readingDown;
      if (currentDownState == LOW) {
        handleDownButton();
      }
    }
  }
  lastDownState = readingDown;

  // --- RESET BUTTON ---
  bool readingReset = digitalRead(RESET_BUTTON);
  if (readingReset != lastResetState) {
    lastResetDebounceTime = currentMillis;
  }
  if ((currentMillis - lastResetDebounceTime) > DEBOUNCE_DELAY) {
    if (readingReset != currentResetState) {
      currentResetState = readingReset;
      if (currentResetState == LOW) {
        handleResetButton();
      }
    }
  }
  lastResetState = readingReset;

  // --- SMART EEPROM AUTO-SAVE ---
  if (needsSaving && (currentMillis - lastButtonPressTime >= EEPROM_SAVE_DELAY)) {
    int eepromCheck;
    EEPROM.get(eepromAddress, eepromCheck);
    if (eepromCheck != count) {
      EEPROM.put(eepromAddress, count);
    }
    needsSaving = false;
  }
}

// --- BUTTON ACTION ROUTINES ---
void handleUpButton() {
  lastButtonPressTime = millis();
  count++;
  if (count > 9999) {
    count = 0;
  }
  needsSaving = true;
  updateDisplay();
}

void handleDownButton() {
  lastButtonPressTime = millis();
  count--;
  if (count < 0) {
    count = 9999;
  }
  needsSaving = true;
  updateDisplay();
}

void handleResetButton() {
  lastButtonPressTime = millis();
  count = 0;
  
  // Safe write behavior: check before burning an EEPROM rewrite cycle
  int eepromCheck;
  EEPROM.get(eepromAddress, eepromCheck);
  if (eepromCheck != count) {
    EEPROM.put(eepromAddress, count); 
  }
  
  needsSaving = false;             
  updateDisplay();
}

// --- DISPLAY DRIVER ROUTINE ---
void updateDisplay() {
  display.showNumberDec(count, true);
}
