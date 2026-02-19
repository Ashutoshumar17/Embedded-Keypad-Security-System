## 📷 Circuit Diagram

<img width="1536" height="632" alt="Fantastic Kasi" src="https://github.com/user-attachments/assets/f4a387ab-0287-4b80-9e68-dc05415b8d95" />


## Code

#include <Keypad.h>
#include <LiquidCrystal.h>
#include <Servo.h>
#include <string.h>

#define PASSWORD_LENGTH 5
#define MAX_ATTEMPTS 3


Servo myservo;
LiquidCrystal lcd(A0, A1, A2, A3, A4, A5);


const byte Rows = 4;
const byte Cols = 4;

char keys[Rows][Cols] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPin[Rows] = {2, 3, 4, 5};
byte colPin[Cols] = {6, 7, 8, 9};

Keypad keypad = Keypad(makeKeymap(keys), rowPin, colPin, Rows, Cols);

char masterPassword[PASSWORD_LENGTH + 1] = "A5765";
char inputPassword[PASSWORD_LENGTH + 1];

byte indexCount = 0;
byte wrongAttempts = 0;


void setup()
{
  myservo.attach(11);
  myservo.write(0);   // Door locked

  lcd.begin(16, 2);
  lcd.clear();
  lcd.print("HOME SECURITY");
  lcd.setCursor(0,1);
  lcd.print("System Ready");
  delay(2000);
  lcd.clear();

  showPrompt();
}


void loop()
{
  char key = keypad.getKey();

  if (key)
  {
    handleKey(key);
  }
}


void handleKey(char key)
{
  if (key == '*')       
  {
    clearInput();
    lcd.clear();
    showPrompt();
  }
  else if (key == '#')  
  {
    checkPassword();
  }
  else
  {
    if (indexCount < PASSWORD_LENGTH)
    {
      inputPassword[indexCount] = key;
      lcd.setCursor(indexCount,1);
      lcd.print("*");   
      indexCount++;
    }
  }
}

// -------- CHECK PASSWORD --------
void checkPassword()
{
  inputPassword[indexCount] = '\0';

  if (strcmp(inputPassword, masterPassword) == 0)
  {
    accessGranted();
  }
  else
  {
    accessDenied();
  }

  clearInput();
}


void accessGranted()
{
  wrongAttempts = 0;

  lcd.clear();
  lcd.print("Access Granted");
  delay(1000);

  openDoor();

  lcd.clear();
  lcd.print("Door Opened");
  delay(3000);

  closeDoor();

  lcd.clear();
  showPrompt();
}


void accessDenied()
{
  wrongAttempts++;

  lcd.clear();
  lcd.print("Wrong Password");
  delay(1500);

  if (wrongAttempts >= MAX_ATTEMPTS)
  {
    lcd.clear();
    lcd.print("SYSTEM LOCKED");
    lcd.setCursor(0,1);
    lcd.print("Wait 10 sec");
    delay(10000);
    wrongAttempts = 0;
  }

  lcd.clear();
  showPrompt();
}


void openDoor()
{
  for (int pos = 0; pos <= 90; pos += 5)
  {
    myservo.write(pos);
    delay(20);
  }
}


void closeDoor()
{
  for (int pos = 90; pos >= 0; pos -= 5)
  {
    myservo.write(pos);
    delay(20);
  }
}


void clearInput()
{
  for (int i = 0; i < PASSWORD_LENGTH; i++)
  {
    inputPassword[i] = 0;
  }
  indexCount = 0;
}


void showPrompt()
{
  lcd.setCursor(0,0);
  lcd.print("Enter Password:");
}
