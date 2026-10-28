# CMPM121_S04_CodeSmells

Jade Gardner, Cassidy Croy-Perrett, Trinity Willis

Smells chosen: Data Clumps (Cassidy), Primitive Obsession (Trinity), Large Class (Jade)

# Data Clumps

## Introduction
Data Clumps refer to a means of ineffienciently passing data between functions, with related items being often sent to the same places without being properly grouped together under a singular hood. This can lead to code that is increasingly difficult to read and write (for example, should a function have an exroadinary amount of required inputs, it may be a hassle keeping straight which similar & related piece of data goes into what order.) Not to 

## Refactoring
Data Clumps may be refactored through the bundling related pieces of data (whose usage often goes together in the program) under a single, more comprehensive class object. For example, instead of passing around an objects x, y and z coordinates in a 3D space around seperately, grouping them together into a singular "position" class object (where the variables can be referred to as position.x, position.y and position.z respectivly), and passing that object to related functions, can increase the ease and speed that functions may be written that involve passing these pieces of data from function to function.

### Javascript (Phaser) Example:

Listed below is an excerpt of code that I wrote for a project in a precious course. This Simulation involved the growing of different types of plants within a grassy field, containing both a location within the world and a position of the plant's current sprite within the spritesheet. Note the rather chaotic and un-sightreadable calling of the new Crop() constructor within the addCropSprite() function.


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
    }

We may refactor this code through the bundling of related datatypes together, creating aptly labeled objects to contain them.

    class Crop extends Phaser.GameObjects.Sprite {  

        constructor(scene, position, spritesheet) {  

            super(scene, position.x, position.y, spritesheet.textureList[spritesheet.frame], null);  

            this.stages = spritesheet.textureList;  

            this.scene = scene;  

            scene.add.existing(this);
            return this;
        }
        remove() {
            this.destroy();
        }
    }
  

    addCropSprite(u, v) {  
        let position = {x: u * this.tileWidth + this.tileWidth / 2, 
        y: v * this.tileHeight + this.tileHeight * 3 / 4};

        let spritesheet = {texturelist: plantTypes[1].growthFrames, frame: 0}

        let newCrop = new Crop(this, position, spritesheet);  
    }

With this refactoring, it is much easier to read the function and form of the Crop constructor, and build new callings of this function in different circumstances at a moment's notice.

### C-Sharp (Unity) Example:
Detailed here is a section of code that I wrote for a previous Unity-based game project. WHile Unity's gameobject system does a fine job automatically grouping related components, data clumps can still form within the database if not careful. In this specific project, the gameplay was based on the spawning and aiming of genade objects, the resulting explosion launching the player to complete platforming challenges and knocking out enemies. Below is an Ienumerator (function that completes it's task over time) that handled the animation and physics of the grendate's explosion.

     private IEnumerator Boom()
    {
        gameObject.GetComponent<SpriteRenderer>().enabled = false;
        blast.GetComponent<Blast>().enabled = true;
        blast.GetComponent<SpriteRenderer>().enabled = true;
        blast.GetComponent<Collider2D>().enabled = true;
        gameObject.GetComponent<AudioSource>().Play();
        yield return new WaitForSeconds(0.5f);

        //time deference between "appearing" deleted and being deleted for grenade refill delay
        blast.GetComponent<Blast>().enabled = false;
        blast.GetComponent<SpriteRenderer>().enabled = false;
        blast.GetComponent<Collider2D>().enabled = false;
        transform.Find("Canvas").gameObject.SetActive(false);
        gameObject.GetComponent<SpriteRenderer>().enabled = false;
        gameObject.GetComponent<Collider2D>().enabled = false;

        yield return new WaitForSeconds(0.1f);
        GameObject.Find("Player").GetComponent<Player_Move>().grenades -= 1;
        //Destroy(gameObject);
    }

While manually toggling each individual component of the genades blast (it's sprite, script and collider objects) work, it is not sight-readable and clogs up the code. By seperating these specific aspects of the blast gameobject to be under a child gameobject, these specific behaviors can be more easily accessed by a singular command.

    private IEnumerator Boom(){
        gameObject.GetComponent<SpriteRenderer>().enabled = false;
        blastChild.GetComponent<BlastCode>().startExploding();
        gameObject.GetComponent<AudioSource>().Play();
        yield return new WaitForSeconds(0.5f);

        //time deference between "appearing" deleted and being deleted for grenade refill delay
        blastChild.GetComponent<BlastCode>().stopExploding();
        transform.Find("Canvas").gameObject.SetActive(false);
        gameObject.GetComponent<SpriteRenderer>().enabled = false;
        gameObject.GetComponent<Collider2D>().enabled = false;

        yield return new WaitForSeconds(0.1f);
        GameObject.Find("Player").GetComponent<Player_Move>().grenades -= 1;
        //Destroy(gameObject);
    }

By grouping those aspects of the origional gameobject under a seperate child  with it's own script, these particular functions can be more easily accessed and enabled on command all together.

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

![picture showing class TestNight with 739 lines](/images/longTestNightClass.png)
Here is an example of a very long class from my final project in CMPM 121. With the use of Phaser it is a bit hard to use some of Fowler's refactoring methods as described.
![picture showing how many night classes exist is the sample project](/images/multipleNights.png)
However, the same basic idea of a night class gets used repeatedly so it makes sense to refactor this.


![picture showing TestNight refactored down to 678 lines](/images/refactoredTestNight.png)
By pulling out some behaviors using **Extract Function (106).**, including the creation of door objects as depicted here, I can refactor the class down to 678 lines.

However, to lose the repeated code in each night we can create one base class out of the TestNight code. I called this class nightBase in my example. Then we can just call each function from the base class. Things that vary from night to night are held in a JSON file which we could create an object for and pass into the functions where necessary. This creates one night class of 678 lines (which could probably be further reduces if I understood what Phaser allows) and all the rest of the nights only have 36 lines

        class nightTwo extends nightBase{
        constructor() {
            super("nightTwoScene");
        }

        init() {
            //initaling all the variables from nightBase class
            this.init();
        }

        create() {
            //creating all the objects from nightBase class
            this.enemyD = this.cache.json.get('enemiesNight2');
            this.create(enemyD)
        }

        update(time, delta) {
            this.update(time, delta);
        
        }

        updatePower() {
            this.updatePower();
        }

        // called when 6 AM is reached
        hasWon() {
            this.hasWon();
        }

        // creates new button sprite and adds it to buttons
        createCamButton(x, y, sprite, scale) {
            this.createCamButton(x, y, sprite, scale);
        }
    }
    


### Example 2 (Java):

    package gameoflife;

    import java.awt.Color;
    import java.awt.Graphics;

    public class Game extends Life{
	
	    Life[][] gameboard;
	    int state;
	    int alive = 0;
	    //Graphics g;
	
	Game(int state, int r, int c){
		    super(state);
		    this.state = state;
		    gameboard=new Life[r][c];
        for(int j=0;j<gameboard.length;j++) {
    	    for(int k=0;k<gameboard[0].length;k++){
    		    double col=Math.random();double sh=Math.random();
    		    if(col<=.4&&sh<=.6) {gameboard[j][k]=new Life(1);}
    		    if(col<=.4&&sh>.6) {gameboard[j][k]=new Life(1);}
    		    if(col>.4&&sh<=.6) {gameboard[j][k]=new Life(0);}
    		    if(col>.4&&sh>.6) {gameboard[j][k]=new Life(0);}
    		}
    	}
	}
	
	public void prtBoard() {
		for(int j=0;j<gameboard.length;j++) {
			for(int k=0;k<gameboard[0].length;k++) {
				if(gameboard[j][k].state==1) {System.out.print("1 ");}
				else if(gameboard[j][k].state==0) {System.out.print("0 ");}
			}	
			System.out.println("");
		}
	}
	
	 void paintBoard(int len, Graphics g) {
	  	  for(int j=0;j<this.gameboard.length;j++) {
	  		  for(int k=0;k<this.gameboard[0].length;k++) {
	  			  gameboard[j][k].draw(j,k,len,g);
	  		  }
	  	  }
	}
	
	public void playGame() {
		for(int j=1;j<gameboard.length-1;j++) {
			for(int k=1;k<gameboard[0].length-1;k++) {
			if(gameboard[j-1][k-1].state==1) {alive++;}
			if(gameboard[j-1][k].state==1) {alive++;}
			if(gameboard[j-1][k+1].state==1) {alive++;}
			if(gameboard[j][k-1].state==1) {alive++;}
			if(gameboard[j][k+1].state==1) {alive++;}
			if(gameboard[j+1][k-1].state==1) {alive++;}
			if(gameboard[j+1][k].state==1) {alive++;}
			if(gameboard[j+1][k+1].state==1) {alive++;}
			
			    if(alive>3 && gameboard[j][k].state==1) {gameboard[j][k] = new Life(0);alive--;}
			    else if(alive<2 && gameboard[j][k].state==1) {gameboard[j][k] = new Life(0);alive--;}
			    else if(alive==3 && gameboard[j][k].state==0) {gameboard[j][k] = new Life(1); alive++;}
			    alive = 0;
			}
		}
		}
	
	    public static void main(String args[]) {
		    Game g1 = new Game(0,10,10);
		    g1.prtBoard();
		    g1.playGame();
		    System.out.println("");
		    System.out.println("");
	    }
    }

In this example the game board is being printed in the same class as the gameplay takes place in. 

    package gameoflife;

    import java.awt.Color;
    import java.awt.Graphics;
    import gameoflife.Play;
    import gameoflife.Board;

    public class Game extends Life{
	
	    Life[][] gameboard;
	    int state;
	    int alive = 0;
	    //Graphics g;
	
	Game(int state, int r, int c){
		    super(state);
		    this.state = state;
		    gameboard=new Life[r][c];
        for(int j=0;j<gameboard.length;j++) {
    	    for(int k=0;k<gameboard[0].length;k++){
    		    double col=Math.random();double sh=Math.random();
    		    if(col<=.4&&sh<=.6) {gameboard[j][k]=new Life(1);}
    		    if(col<=.4&&sh>.6) {gameboard[j][k]=new Life(1);}
    		    if(col>.4&&sh<=.6) {gameboard[j][k]=new Life(0);}
    		    if(col>.4&&sh>.6) {gameboard[j][k]=new Life(0);}
    		}
    	}
	}
	
	    public static void main(String args[]) {
		    Game  g1 = new Game(0,10,10);
		    prtBoard(g1);
		    playGame(g1);
	        System.out.println("");
	        System.out.println("");
	    }
    }

To refactor this I use **Extract Class (182)** to separate the print behaviors and the play behaviors. Those classes hold the same behaviors as before but not take a game object. Also upon further inspect the method paintBoard() was not being utilized so it can be removed from the class.

     package gameoflife;

    import java.awt.Color;
    import java.awt.Graphics;

     public class Play extends Life{

     gameBoard = Life[][];

     Play(Game g){
         this.gameboard = g.gameBoard;
     }

     public void playGame() {
		for(int j=1;j<gameboard.length-1;j++) {
			for(int k=1;k<gameboard[0].length-1;k++) {
			if(gameboard[j-1][k-1].state==1) {alive++;}
			if(gameboard[j-1][k].state==1) {alive++;}
			if(gameboard[j-1][k+1].state==1) {alive++;}
			if(gameboard[j][k-1].state==1) {alive++;}
			if(gameboard[j][k+1].state==1) {alive++;}
			if(gameboard[j+1][k-1].state==1) {alive++;}
			if(gameboard[j+1][k].state==1) {alive++;}
			if(gameboard[j+1][k+1].state==1) {alive++;}
			
			    if(alive>3 && gameboard[j][k].state==1) {gameboard[j][k] = new Life(0);alive--;}
			    else if(alive<2 && gameboard[j][k].state==1) {gameboard[j][k] = new Life(0);alive--;}
			    else if(alive==3 && gameboard[j][k].state==0) {gameboard[j][k] = new Life(1); alive++;}
			    alive = 0;
			}
		}
	}
    }

This new version of the play class would look something like this. The board class would function in the same way just handling the printing behavior.




