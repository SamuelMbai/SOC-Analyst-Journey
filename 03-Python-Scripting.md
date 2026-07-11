Python Scripting  

---

## Task 1: Introduction to python
Syntax is important as it describes the structure of a valid programming language; syntax tells the computer how to read code using rules that control the structure of symbols, punctuation, and words of a programming language.

### Answer the questions below
Run what's currently in the code editor by clicking the green "Run Code" button (on the right-hand side of your screen), and move onto the next task.
*Completed*

---

## Task 2: Hello world
We can control what is output to the screen by using the print() statement. Anything inside of the parenthesis () will be output.

### Answer the questions below
On the code editor, print "Hello World". What is the flag?  
**Answer:** `THM{PRINT_STATEMENTS}`

---

## Task 3: Mathematical Operations
### Answer the questions below

1. In the code editor, print the result of 21 + 43. What is the flag?  
   **ANSWER:** `THM{ADDITION}`
2. Print the result of 142 - 52. What is the flag?  
   **ANS:** `THM{SUBTRCT}`
3. Print the result of 10 * 342. What is the flag?  
   **ANSWER:** `THM{MULTIPLICATION_PYTHON}`
4. Print the result of 5 squared. What is the flag?  
   **ANSWER:** `THM{EXP0N3NT_POWER}`

---

## Task 4: Variables and data types
Variables allow you to store and update data in a computer program. You have a variable name and store data to that name.

**Data Types:** is the type of data being stored in a variable. You can store text, or numbers, and many other types. The data types to know are:
* **String** - Used for combinations of characters, such as letters or symbols
* **Integer** - Whole numbers
* **Float** - Numbers that contain decimal points or for fractions
* **Boolean** - Used for data that is restricted to True or False options
* **List** - Series of different data types stored in a collection

| Data Type | String | Float | Integer | Boolean | List |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Title** | Star Wars | Matrix | Indiana Jones | | |
| **Rating** | 9.8 | 8.5 | 6.1 | | |
| **Times Viewed** | 13 | 23 | 3 | | |
| **Favorite** | True | False | False | | |
| **Seen By** | Alice, Bob | Charlie | Daniel, Evie | | |

### Answer the questions below
In the code editor, create a variable called height and set its initial value to 200.  
On a new line, add 50 to the height variable.  
On another new line, print out the value of height. What is the flag that appears?  
**Answer:** `THM{VARIABL3S}`

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

In shipping.py, on line 15 (when using the Code Editor's Hint), change the customer_basket_cost variable to 101 and re-run your code. You will get a flag (if the total cost is correct based on your code); the flag is the answer to this question.  
**ANSWER:** `THM{MY_FIRST_APP}`

---

## Task 7: Loops
Loops allow programs to iterate and perform actions a number of times. There are two types of loops, for and while loops.

### While Loops
```python
i = 1
while i <= 10:
    print(i)
    i = i + 1
