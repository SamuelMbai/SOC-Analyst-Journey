Python Scripting  

---

## Task 1: Introduction to python
Syntax is important as it describes the structure of a valid programming language; syntax tells the computer how to read code using rules that control the structure of symbols, punctuation, and words of a programming language.

### Answer the questions below
Run what's currently in the code editor by clicking the green "Run Code" button (on the right-hand side of your screen), and move onto the next task.
<img width="806" height="452" alt="image" src="https://github.com/user-attachments/assets/f80407c7-623c-4289-873b-68b284790d1c" />


---

## Task 2: Hello world
We can control what is output to the screen by using the print() statement. Anything inside of the parenthesis () will be output.
<img width="891" height="503" alt="image" src="https://github.com/user-attachments/assets/95e4b99e-b869-4d60-b172-48237d0da598" />


### Answer the questions below
On the code editor, print "Hello World". What is the flag?  
**Answer:** `THM{PRINT_STATEMENTS}`
<img width="891" height="503" alt="image" src="https://github.com/user-attachments/assets/63025e30-f7e7-4b6e-91d1-5734c7cdee55" />



---

## Task 3: Mathematical Operations
### Answer the questions below

1. In the code editor, print the result of 21 + 43. What is the flag?  
   **ANSWER:** `THM{ADDITION}`
   <img width="948" height="537" alt="image" src="https://github.com/user-attachments/assets/776127e1-ef1a-47a2-bb1d-97f69b720929" />

3. Print the result of 142 - 52. What is the flag?  
   **ANS:** `THM{SUBTRCT}`
   <img width="965" height="543" alt="image" src="https://github.com/user-attachments/assets/7d19fdfe-2419-46ee-9fe6-752fb2e21627" />

4. Print the result of 10 * 342. What is the flag?  
   **ANSWER:** `THM{MULTIPLICATION_PYTHON}`
   <img width="934" height="465" alt="image" src="https://github.com/user-attachments/assets/80bb2fd1-e945-4951-b7a0-a83c9b9d9975" />

6. Print the result of 5 squared. What is the flag?  
   **ANSWER:** `THM{EXP0N3NT_POWER}`
   <img width="810" height="454" alt="image" src="https://github.com/user-attachments/assets/29bdf8fc-7d61-455b-9c5a-081800d9c4d7" />


---

## Task 4: Variables and data types
Variables allow you to store and update data in a computer program. You have a variable name and store data to that name.

**Data Types:** is the type of data being stored in a variable. You can store text, or numbers, and many other types. The data types to know are:
* **String** - Used for combinations of characters, such as letters or symbols
* **Integer** - Whole numbers
* **Float** - Numbers that contain decimal points or for fractions
* **Boolean** - Used for data that is restricted to True or False options
* **List** - Series of different data types stored in a collection

### Answer the questions below
In the code editor, create a variable called height and set its initial value to 200.  
On a new line, add 50 to the height variable.  
On another new line, print out the value of height. What is the flag that appears?  
**Answer:** `THM{VARIABL3S}`

<img width="797" height="446" alt="image" src="https://github.com/user-attachments/assets/052b86e0-6901-403d-b588-be392e828983" />


---

## Task 5: Logical and Boolean Operators
Logical operators allow assignment and comparisons to be made and are used in conditional testing (such as if statements).

### Logical Operation
| Operator | Example |
| :--- | :--- |
| Equivalence | `if x == 5` |
| Less than | `if x < 5` |
| Less than or equal to | `if x <= 5` |
| Greater than | `if x > 5` |
| Greater than or equal to | `if x >= 5` |

Boolean operators are used to connect and compare relationships between statements. Like an if statement, conditions can be true or false.

### Boolean Operation
| Boolean Operation | Operator | Example |
| :--- | :--- | :--- |
| Both conditions must be true for the statement to be true | **AND** | `if x >= 5 AND x <= 10` Returns TRUE if x is a number between 5 and 10 |
| Only one condition of the statement needs to be true | **OR** | `if x == 1 OR x == 10` Returns TRUE if X is 1 or 10 |
| If a condition is the opposite of an argument | **NOT** | `if NOT y` Returns TRUE if the y value is False |

---

## Task 6: Shipping Project: Introduction to If Statements
Using "if statements" allows programs to make decisions.

### Answer the questions below
In this exercise, we will code a small application that calculates and outputs the shipping cost for a customer based on how much they've spent.  
In the code editor, click on the "shipping.py" tab and follow the instructions to complete this task.  
**ANSWER:** `THM{IF STATEMENT SHOPPING}`
<img width="795" height="447" alt="image" src="https://github.com/user-attachments/assets/9992715f-5c07-4af6-807c-3d28b2ce986f" />


In shipping.py, on line 15 (when using the Code Editor's Hint), change the customer_basket_cost variable to 101 and re-run your code. You will get a flag (if the total cost is correct based on your code); the flag is the answer to this question.  
**ANSWER:** `THM{MY_FIRST_APP}`
<img width="850" height="450" alt="image" src="https://github.com/user-attachments/assets/5dae6ae2-049d-471c-abfc-b302dc8a9117" />


---

## Task 7: Loops
Loops allow programs to iterate and perform actions a number of times. There are two types of loops, for and while loops.

### While Loops
```python
i = 1
while i <= 10:
    print(i)
    i = i + 1

For Loops
A for loop is used to iterate over a sequence such as a list. Lists are used to store multiple items in a single variable, and are created using square brackets.

websites = ["facebook.com", "google.com", "amazon.com"]
for site in websites:
    print(site)

Answer the questions below
On the code editor, click back on the "script.py" tab and code a loop that outputs every number from 0 to 50.
<img width="801" height="426" alt="image" src="https://github.com/user-attachments/assets/46101b95-4962-4ec3-8507-b39c8fbaf571" />



Task 8: introduction to functions
A function is a block of code that can be called at different places in your program. Having functions removes repetitive code, as the function's purpose can be used multiple times throughout a program.

def sayHello(name):
    print("Hello " + name + "! Nice to meet you.")

sayHello("sam") # Output is: Hello sam! Nice to meet you



Answer the questions below
You've invested in Bitcoin and want to write a program that tells you when the value of Bitcoin falls below a particular value in dollars.

In the code editor, click on the bitcoin.py tab. Write a function called bitcoinToUSD with two parameters: bitcoin_amount, the amount of Bitcoin you own, and bitcoin_value_usd, the value of bitcoin in USD. The function should return usd_value, which is your bitcoin value in USD (to calculate this, in the function, you times bitcoin_amount variable by bitcoin_value_usd variable and return the value). The start of the function should look like this:

def bitcoinToUSD(bitcoin_amount, bitcoin_value_usd):


Once you've written the bitcoinToUSD function, use it to calculate the value of your Bitcoin in USD, and then create an if statement to determine if the value falls below $30,000; if it does, output a message to alert you (via a print statement).

ANSWER: THM{BITCOIN_INVESTOR}
<img width="809" height="404" alt="image" src="https://github.com/user-attachments/assets/bccc3fab-9e76-48e5-b3b2-c2ef3fa39485" />




TASK 9: Files
To open the file, we use the built-in open() function, and the "r" parameter stands for "read" and is used as we're reading the contents of the file.

Python
f = open("file_name", "r")
print(f.read())
You can also create and write files. If you're writing to an existing file, you open the file first and use the "a" in the open function after the filename call (which stands for append). If you're writing to a new file, you use "w" (write) instead of "a".

Python
f = open("demofile1.txt", "a") # Append to an existing file
f.write("The file will include more text..")
f.close()

f = open("demofile2.txt", "w") # Creating and writing to a new file
f.write("demofile2 file created, with this content in!")
f.close()

Answer the questions below
In the code editor, write Python code to read the flag.txt file. What is the flag in this file?

ANSWER: THM{FILE_R3AD}
<img width="884" height="469" alt="image" src="https://github.com/user-attachments/assets/90bfae42-958e-4dcc-915b-cb7937ac5e2e" />



Task 10: Imports
We import other libraries using the import keyword. Then in Python, we use that import's library name to reference its functions. In the example below, we import datetime, then access the .now() method by calling library_name.method_name().

Python
import datetime
current_time = datetime.datetime.now()
print(current_time)



---

## Conclusion

This assignment provided a practical foundation in Python scripting, advancing from syntax basics to structural coding techniques. Through hands-on exercises, key milestones were met across essential domains:

* **Core Mechanics:** Mastered printing, basic arithmetic, variable management, and data types (Strings, Integers, Floats, Booleans, and Lists).
* **Logic & Control Flow:** Implemented conditional execution using logical and Boolean operators, allowing programs to make automated decisions.
* **Automation & Modularity:** Leveraged `for` and `while` loops to handle repetitive data tasks, and created reusable functions to pass parameters and return values.
* **System Interaction:** Covered external file operations (reading and writing data) and extended the language's capabilities by importing external libraries.

Completing all 10 tasks successfully bridges the gap between understanding foundational syntax and writing scripts for basic automation.

