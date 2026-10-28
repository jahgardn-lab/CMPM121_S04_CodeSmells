# CMPM121_S04_CodeSmells

Jade Gardner, Cassidy Croy-Perrett, Trinity Willis

Smells chosen: Data Clumps, Primitive Obsession, Large Class


Data Clumps Examples:

    class Crop extends Phaser.GameObjects.Sprite {  

        constructor(scene, x, y, textureList, frame) {  

            super(scene, x, y, textureList[frame], null);  

            this.stages = textureList;  

            this.scene = scene;  

            scene.add.existing(this);
            return this;
        }
        remove() {
            this.destroy();
        }
    }
  

    addCropSprite(u, v) {  

        let newCrop = new Crop(this, u * this.tileWidth + this.tileWidth / 2, 
            v * this.tileHeight + this.tileHeight * 3 / 4, plantTypes[1].growthFrames, 0);  

        let plantScale = (this.tileWidth) / (newCrop.width * 2);  

        newCrop.setScale(plantScale, plantScale);  

        newCrop.setVisible(false);  

        this.cropSprites[[u,v].toString()] = newCrop;  

    }


# Primitive Obsession

## Introduction
Primitive Obsession occurs when programmers rely too heavily on primitive data types and are hesitant to create custom types for specific and reoccurring uses.

A lot of programmers fall into a trap of starting with primitive data types to represent complex ideas like time, money, coordinates, ranges, or phone numbers. While this can work, it becomes highly inefficient when specific mathematical notation and formatting is required to represent these ideas in different sections of the code. This can lead to many repeated instances of similar code that could easily be condensed with a custom data type for these situations. Primitive Obsession can also lead to bugs where invalid data is passed in, for example, a number with 3 decimal points passed in to a variable representing money. A primitive data type does not check whether data is in the right format for its label, but a custom data type could ensure that it only holds valid data according to what it represents.

## Prevention and Refactoring
### Replace Primitive with Object  
Remove the primitive variable and replace it in a small, dedicated class or object.
1. Create a new class
2. Add a constructor, store the primitive as a field inside the new class, and create a getter
3. Move any related code outside the class into the new class's constructor
4. Replace instances of the primitive data type with the new object type
5. Adjust constructors, getters, and setters to accept and return the new object

### Extract Class
Very similar in this context to Replace Primitive with Object. Split a very large class that is managing many unrelated things by removing groups of code related to processing and formatting a primitive data type into its own class.
1. Identify all the code in one class related to a single primitive data type
2. Create a new class
3. Move relevant code to the new class
4. Update code in the original class to use the new class instead of the primitive data type

### Replace Type Code with Subclasses
When code dictates how an object behaves but does not change after the object is created, replace that primitive data type with a class hierarchy. The overarching type becomes an abstract base class and the previous values of the primitive data type become their own subclasses.
1. Use Encapsulate Variable to wrap a primitive data type in its own class
2. Create a subclass for each potential value of the primitive data type
3. Write a constructor that uses the values of the primitive data type to instiantiate the correct subclass
4. Remove the primitive variable

### Replace Conditional with Polymorphism
Following Replace Type Code with Subclasses, conditional statements that previously filtered the primitive data type can be removed.
1. Find the conditionals that processed the primitive data type
2. Move the conditional code into its corresponding subclass
3. Make a corresponding method in the parent class abstract
4. Replace the conditional calls with direct method calls on the object

## Code Examples

### Java

Before Replace Primitive with Object 

        double price = 19.99;

        System.out.printf("Price: $%.2f%n", price);

Here, every time the price variable is printed, it needs to be formatted specifically to two decimal points to adhere to standard formatting for money.

After Replace Primitive with Object 

        class Money {
            private double amount;

            public Money(double amount) {
                this.amount = amount;
            }

            public double getAmount() {
                return amount;
            }

            public void print() {
                System.out.printf("Price: $%.2f%n", amount);
            }
        }

        // Usage
        Money price = new Money(19.99);
        price.print();

After refractoring with Replace Primitive with Object, calling print() on any Money object automatically formats it to standard 2 decimal formatting.

### JavaScript

Before Replace Type Code with Subclasses and Replace Conditional with Polymorphism

        class Employee {
            constructor(type, hours) {
                this.type = type;
                this.hours = hours;
            }

            calculatePay() {
                if (this.type === "regular") {
                    return this.hours * 20;
                } else if (this.type === "overtime") {
                    return this.hours * 30;
                }
            }
        }

        //Usage
        const employee = new Employee("overtime", 10);
        console.log(employee.calculatePay()); // 300

Here the employee's pay type is represented by a string. The conditional statement checks whether the employee is paid for regular time or over time to do pay calculations.

After Replace Type Code with Subclasses and Replace Conditional with Polymorphism

        class Employee {
            constructor(hours) {
                this.hours = hours;
            }

            calculatePay() {
                throw new Error("Subclass must implement calculatePay()");
            }
        }

        class RegularEmployee extends Employee {
            calculatePay() {
                return this.hours * 20;
            }
        }

        class OvertimeEmployee extends Employee {
            calculatePay() {
                return this.hours * 30;
            }
        }

        //Usage
        const employee = new OvertimeEmployee(10);
        console.log(employee.calculatePay()); // 300

Here, the Employee class becomes a parent class for new subclasses RegularEmployee and OvertimeEmployee. Each subclass has the function calculatePay() which runs the corresponding pay calculation for each type of employee.




# Large Class

Large class code smell is similar to other smells like large function. When a class gets large there is often repeated code or too many ideas shoved into one class.

## Refactoring

Fowler suggests a couple different refactoring method for large class. You can **Extract Class (182)** to bundle specific variables and methods together. If your class could use inheritance you can also use **Extract Superclass (375)** or you can replace some code with a subclass and use **Replace Type Code
with Subclasses (362)**. You may also have to apply other kinds of refactoring that might be related to smells like Large Functions or other similar smells that might contribute to the size of your class.

### Example 1 (Javascript):

![picture showing class TestNight with 739 lines](/Screenshot 2026-10-27 173650.png)
![picture showing how many night classes exist is the sample project](/CMPM121_S04_CodeSmells
/Screenshot 2026-10-27 180158.png)

