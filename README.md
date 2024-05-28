# Activities
1. dip-eip-singleton-and-template-method
2. dip-eip-composite-strategy-and-iterator
3. dip-eip-message-queue
4. dip-eip-layers-facade-strategy
5. camel-software-setup
6. camel-producer-subscriber
7. camel-message-from-rss

# Singleton Pattern & Template Method Pattern
# Set up the lab
This project was built using Java version 21.0.2. Having it installed, you can run the `App.java` file from the main directory to get the expected results.

This project also uses Apache Maven 3.9.6 to run further unittests in order to check the behaviour of specific classes. Having it installed, run the following commands:

```bash
mvn compile
```

```bash
mvn install
```

# Structure
To properly deliver the homework, a Maven project is used.

- All the classes are built inside the `src/main` folder.
- All the unittests are built inside the `src/test` folder.

To run the unittests, run:
```
mvn test
```

A main file is also built with the specifications provided in the exercise question. It is called `App.java`, also inside `src/main/java/lab1/group` folder.

To run `App.java` you should NOT go inside its folder. It should be run from the main folder.

```
.
├── README.md
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── lab1
    │           └── group
    │               ├── Account.java
    │               ├── App.java
    │               ├── Bank.java
    │               ├── CheckingsAccount.java
    │               └── SavingsAccount.java
    └── test
        └── java
            └── lab1
                └── group
                    ├── AppTest.java
                    ├── BankTest.java
                    ├── SingletonTest.java
                    └── TemplateMethodTest.java
```

# Bank Implementation
The `Bank` class is implemented as a Singleton:

```java
package lab1.group;
import java.util.ArrayList;
import java.util.List;

class Bank {
    private Bank() {}

    public static Bank getBank() {
        if (instance == null) {
            synchronized(Bank.class) {
                if (instance == null) {
                    instance = new Bank();
                }
            }
        }
        return instance;
    }

    public void addAccount(Account account) {
        accounts.add(account);
    }

    public void showAccounts() {
        System.out.println("\nBANK ACCOUNTS:");
        if (accounts.size() == 0) {
            System.out.println("NO ACCOUNTS FOUND");
        } else {
            for (int i = 0; i < accounts.size(); i++) {
                int n = i + 1;
                System.out.println("(" + n + ") " + accounts.get(i).toString());
            }
        }
        System.out.println("\n");
        return;
    }

    public List<Account> getAccounts() {
        return accounts;
    }

    private List<Account> accounts = new ArrayList<Account>();
    private static Bank instance;
}
```

The most important aspect of a Singleton is the fact that the constructor is not public:

```java
private Bank() {}
```

Everytime we want to access the Bank, our class checks if an instance already exists or not:

```java
public static Bank getBank() {
    if (instance == null) {
        synchronized(Bank.class) {
            if (instance == null) {
                instance = new Bank();
            }
        }
    }
    return instance;
}
```

# Accounts Implementation
## Account
The class `Account` is a Template Method: it is an abstract class with some methods implemented.

The `Account`:

```java
abstract public class Account {

    public Account(double balance, LocalDate accountCreationDate, String accountName) {
        this.accountName = accountName;
        this.balance = balance;
        this.accountCreationDate = accountCreationDate;
        this.lastCalculationDate = accountCreationDate;
        this.deltaInterest = 0.0;
    }

    // override constructor when the parameter accountName is not provided
    public Account(double balance, LocalDate accountCreationDate) {
        this.balance = balance;
        this.accountCreationDate = accountCreationDate;
        this.lastCalculationDate = accountCreationDate;
        this.deltaInterest = 0.0;
        this.periodsInterest = 0;
    }

    // ...

}
```

The `GetAccountSummary` method is already implemented inside `Account` as well as `UpdateBalance` and `PrintSummary`:

```java
public void GetAccountSummary(LocalDate dateUpdate) {
    CalcInterest(dateUpdate);
    UpdateBalance();
    PrintSummary();
}

private void UpdateBalance(double interestAmount) {
    this.balance += interestAmount;
}

private void PrintSummary() {
    System.out.println("\n" + this.toString() + " SUMMARY");
    System.out.println("Balance: " + this.balance);
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    System.out.println("Last Update: " + lastCalculationDate.format(formatter));
    System.out.println("\n");
}
```

`CalcInterest` is the only method that needs to be implemented in the classes that inherit from the base class:

```java
abstract public double CalcInterest(LocalDate dateUpdate);
```

## Checkings Account
The checkings account does not have any interest rate. Because of that, its implementation is quite simple.

As requested, we implement the `CalcInterest` in the derived class. The method returns the accrued interest amount. Because the checkings account has no interest rate, the interest amount is always zero. We also save `deltaInterest` as an attribute, in case it becomes necessary for the class.

Checkings Account:
```java
package lab1.group;
import java.time.LocalDate;

public class CheckingsAccount extends Account {

    protected double interestRate = 0.00;

    public CheckingsAccount(double balance, LocalDate accountCreationDate, String accountName) {
        super(balance, accountCreationDate, accountName);
    }

    // override constructor when the parameter accountName is not provided
    public CheckingsAccount(double balance, LocalDate accountCreationDate) {
        super(balance, accountCreationDate);
    }
    
    public double CalcInterest(LocalDate dateUpdate) {
        this.deltaInterest = 0.0;
        this.lastCalculationDate = dateUpdate;

        return this.deltaInterest;
    }
}
```

## Savings Account
For savings account, the implementation is a bit more complicated.

When called, `CalcInterest` checks how many january first have passed since the last update.

Afterwards, it compounds the interest rate by the amount of year loops. For instance, if two january first passed since the last update, the total interest accrued would be `balance * (1 + annual interest rate) ^ 2`.

```java
package lab1.group;
import java.time.LocalDate;

public class SavingsAccount extends Account {

    protected double interestRate = 0.01;

    public SavingsAccount(double balance, LocalDate accountCreationDate, String accountName) {
        super(balance, accountCreationDate, accountName);
    }

    // override constructor when the parameter accountName is not provided
    public SavingsAccount(double balance, LocalDate accountCreationDate) {
        super(balance, accountCreationDate);
    }
    
    public void CalcInterest(LocalDate dateUpdate) {
        LocalDate nextJanuaryFirst;

        this.deltaInterest = 0.0;
        this.periodsInterest = 0;

        nextJanuaryFirst = LocalDate.of(this.lastCalculationDate.getYear() + 1, 1, 1);
        while (dateUpdate.isAfter(nextJanuaryFirst)) {
            this.periodsInterest += 1;
            nextJanuaryFirst = LocalDate.of(nextJanuaryFirst.getYear() + 1, 1, 1);
        }

        if (this.periodsInterest > 0) {
            this.deltaInterest = this.getBalance() * (Math.pow(1 + this.interestRate, this.periodsInterest) - 1);
        }

        this.lastCalculationDate = dateUpdate;

        return this.deltaInterest;

    }
}
```

# Implementation Main File
We implement the desired test in `App.java`. Inside, we:
1. Test if `Bank` is a singleton.
2. Create multiple accounts and insert them inside a `Bank`.
3. Update multiple times the balance in different dates.

```java
package lab1.group;
import java.time.LocalDate;

public class App 
{
    public static void main( String[] args ) {

        System.out.println("Test Singleton pattern:");
        Bank b1 = Bank.getBank();
        Bank b2 = Bank.getBank();
        if (b1 == b2) {
            System.out.println("Singleton pattern works");
        } else {
            System.out.println("Singleton pattern failed");
        }

        System.out.println("Test Bank class:");
        b1.showAccounts();
        CheckingsAccount ca1 = new CheckingsAccount(1000.0, LocalDate.of(2020, 1, 1), "Mark Shacklette");
        b1.addAccount(ca1);
        CheckingsAccount ca2 = new CheckingsAccount(1000.0, LocalDate.of(2020, 1, 1), "Alan Salkanovic");
        b1.addAccount(ca2);
        SavingsAccount sa1 = new SavingsAccount(1000.0, LocalDate.of(2020, 1, 1), "Mark Shacklette");
        b1.addAccount(sa1);
        b1.showAccounts();

        System.out.println("Test TemplateMethod pattern:");
        for (int i = 2020; i < 2024; i++) {
            final int year = i;
            b1.accounts.forEach(account -> account.GetAccountSummary(LocalDate.of(year, 6, 1)));
        }

    }
}
```


# Composite, Strategy & Iterator Patterns
# Set up the lab
This project was built using Java version 21.0.2. Having it installed, you can run the `App.java` file from the main directory to get the expected results.

This project also uses Apache Maven 3.9.6 to run further unittests in order to check the behaviour of specific classes. Having it installed, run the following commands:

```bash
mvn compile
```

```bash
mvn install
```

While the main file only follows the specifications defined, the unittests go deeper in the implementation. I strongly recommend that you check the unittets as well.

**PLEASE READ THE README AND THE UNITTESTS TO GET A BETTER PERSPECTIVE OF THE IMPLEMENTATION**

# Structure
To properly deliver the homework, a Maven project is used.

- All the classes are built inside the `src/main` folder.
- All the unittests are built inside the `src/test` folder.

To run the unittests, run:
```bash
mvn test
```

Output:
```
[INFO] -------------------------------------------------------
[INFO] Running lab2.group.MacaulayDurationTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.075 s -- in lab2.group.MacaulayDurationTest
[INFO] Running lab2.group.PortfolioMacaulayDurationTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.019 s -- in lab2.group.PortfolioMacaulayDurationTest
[INFO] Running lab2.group.IteratorTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 s -- in lab2.group.IteratorTest
[INFO] Running lab2.group.PortfolioModifiedDurationTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.009 s -- in lab2.group.PortfolioModifiedDurationTest
[INFO] Running lab2.group.PortfolioTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.004 s -- in lab2.group.PortfolioTest
[INFO] Running lab2.group.ModifiedDurationTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.013 s -- in lab2.group.ModifiedDurationTest
[INFO] Running lab2.group.PresentValueTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.012 s -- in lab2.group.PresentValueTest
[INFO] Running lab2.group.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.012 s -- in lab2.group.AppTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 18, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

A main file is also built with the specifications provided in the exercise question. It is called `App.java`, also inside `src/main/java/lab2/group` folder.

To run `App.java` you should NOT go inside its folder. It should be run from the main folder.

```
.
├── README.md
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── lab2
    │           └── group
    │               ├── AbstractIterator.java
    │               ├── App.java
    │               ├── Asset.java
    │               ├── Bond.java
    │               ├── Duration.java
    │               ├── DurationResult.java
    │               ├── DynamicAssetArray.java
    │               ├── Iterator.java
    │               ├── MacaulayDuration.java
    │               ├── ModifiedDuration.java
    │               └── Portfolio.java
    └── test
        └── java
            └── lab2
                └── group
                    ├── AppTest.java
                    ├── IteratorTest.java
                    ├── MacaulayDurationTest.java
                    ├── ModifiedDurationTest.java
                    ├── PortfolioMacaulayDurationTest.java
                    ├── PortfolioModifiedDurationTest.java
                    ├── PortfolioTest.java
                    └── PresentValueTest.java
```

# Main File
Our main file is the `App.java`. It instanciate the account, which is viewed as a `Asset`, and follows the instructions and gets the desired result:

```java
package lab2.group;

public class App {
    public static void main(String[] args) {
        Bond treasury10Y = new Bond("US10Y", 10000, 10, 1, 10000 * .2 / 1, .04);
        Bond treasury30Y = new Bond("US30Y", 10000, 30, 3, 10000 * .2 / 3, .05);
        Bond fordBond = new Bond("FORD", 10000, 20, 2, 10000 * .2 / 2, .12);
        Portfolio highYieldBonds = new Portfolio("High Yield Bonds", fordBond);
        Portfolio sovereignBonds = new Portfolio("Sovereign Bonds", treasury10Y, treasury30Y);
        Portfolio account = new Portfolio("General", highYieldBonds, sovereignBonds);
        // Printing the account
        System.out.println("Printing Account:");
        System.out.println(account.display());
        System.out.println("Macaulay Duration: " + String.format("%.2f", account.getMacaulayDuration()));
        System.out.println("Modified Duration: " + String.format("%.2f", account.getModifiedDuration()));
        Iterator iterator = account.CreateIterator();
        // Printing the assets
        Asset asset = iterator.First();
        while (!iterator.IsDone()) {
            System.out.println("\n");
            System.out.println(asset.display());
            System.out.println("Macaulay Duration: " + String.format("%.2f", asset.getMacaulayDuration()));
            System.out.println("Modified Duration: " + String.format("%.2f", asset.getModifiedDuration()));
            asset = iterator.Next();
        }
    }
}
```

Output:
```
Printing Account:
General [PORT: 1, BONDS: 3]
Macaulay Duration: 6.11
Modified Duration: 5.89


High Yield Bonds [PORT: 0, BONDS: 0]
Macaulay Duration: 5.52
Modified Duration: 5.21


Sovereign Bonds [PORT: 0, BONDS: 1]
Macaulay Duration: 6.41
Modified Duration: 6.23


BOND FORD [PR: 10000.0, NC: 20, F: 2, V: 1000.00, Y: 12.00%]
Macaulay Duration: 5.52
Modified Duration: 5.21


BOND US10Y [PR: 10000.0, NC: 10, F: 1, V: 2000.00, Y: 4.00%]
Macaulay Duration: 6.60
Modified Duration: 6.34


BOND US30Y [PR: 10000.0, NC: 30, F: 3, V: 666.67, Y: 5.00%]
Macaulay Duration: 6.23
Modified Duration: 6.12
```

I talked to the TA John to validate this format for the account.

# Composite
The composite is build with three main classes:
1. `Asset` (Component)
2. `Portfolio` (Composite)
3. `Bond` (Leaf)

The `Asset` is a abstract class. `Portfolio` and `Bond` extend Asset. We decide to define `Asset` as an abstract class because it follows the approach of the lecture, nonetheless, according to multiple sources a interface would also work well in this design.

Almost all methods (specially the public ones) are shared between the `Portfolio` and the `Bond`, in order to make them as similar as possible to the user. For `Portfolio`, we implement a few other methods to ensure that children can be added (`Bonds`).

## Asset
```java
package lab2.group;

public abstract class Asset {
    abstract public AbstractIterator CreateIterator();
    abstract public DurationResult calculateMacaulayDuration();
    abstract public DurationResult calculateModifiedDuration();
    abstract public Double calculatePresentValue();
    abstract public String display();
    abstract public double getMacaulayDuration();
    abstract public double getModifiedDuration();
}
```

## Portfolio
As we can see below, most of the `Portfolio` methods override `Asset` methods.
```java
package lab2.group;

public class Portfolio extends Asset {
    private String name;
    private Asset[] assets;

    public Portfolio(String name, Asset... assets) {
        this.name = name;
        this.assets = arrangeAssets(assets);
    }

    private Asset[] arrangeAssets(Asset... assets) {
        DynamicAssetArray bondAssets = new DynamicAssetArray();
        DynamicAssetArray portfolioAssets = new DynamicAssetArray();
        for (Asset asset : assets) {
            if (asset instanceof Bond) {
                bondAssets.add(asset);
            } else if (asset instanceof Portfolio) {
                portfolioAssets.add(asset);
            }
        }
        for (int i = 0; i < portfolioAssets.size(); i++) {
            bondAssets.add(portfolioAssets.get(i));
        }
        return bondAssets.toArray();
    }

    @Override
    public Iterator CreateIterator() {
        return new Iterator(this);
    }

    @Override
    public DurationResult calculateMacaulayDuration() {
        double summedMarketValue = 0.0;
        double weightedDuration = 0.0;
        for (Asset asset : this.assets) {
            DurationResult result;
            if (asset instanceof Bond) {
                result = ((Bond) asset).calculateMacaulayDuration();
            } else if (asset instanceof Portfolio) {
                result = ((Portfolio) asset).calculateMacaulayDuration();
            } else {
                continue;
            }
            double newMarketValue = result.presentValue;
            double newDuration = result.duration;
            weightedDuration = ((summedMarketValue * weightedDuration) + (newMarketValue * newDuration)) / (summedMarketValue + newMarketValue);
            summedMarketValue += newMarketValue;
        }
        return new DurationResult(summedMarketValue, weightedDuration);
    }

    @Override
    public double getMacaulayDuration() {
        DurationResult macaulayDurationResult = this.calculateMacaulayDuration();
        return macaulayDurationResult.duration;
    }

    @Override
    public Double calculatePresentValue() {
        Double presentValue = 0.0;
        for (Asset asset : this.assets) {
            presentValue += asset.calculatePresentValue();
        }
        return presentValue;
    }

    @Override
    public DurationResult calculateModifiedDuration() {
        double summedMarketValue = 0.0;
        double weightedDuration = 0.0;
        for (Asset asset : this.assets) {
            DurationResult result;
            if (asset instanceof Bond) {
                result = ((Bond) asset).calculateModifiedDuration();
            } else if (asset instanceof Portfolio) {
                result = ((Portfolio) asset).calculateModifiedDuration();
            } else {
                continue;
            }
            double newMarketValue = result.presentValue;
            double newDuration = result.duration;
            weightedDuration = ((summedMarketValue * weightedDuration) + (newMarketValue * newDuration)) / (summedMarketValue + newMarketValue);
            summedMarketValue += newMarketValue;
        }
        return new DurationResult(summedMarketValue, weightedDuration);
    }

    @Override
    public double getModifiedDuration() {
        DurationResult modifiedDurationResult = this.calculateModifiedDuration();
        return modifiedDurationResult.duration;
    }

    public void addAsset(Asset asset) {
        DynamicAssetArray newAssets = new DynamicAssetArray();
        for (Asset existingAsset : this.assets) {
            newAssets.add(existingAsset);
        }
        newAssets.add(asset);
        this.assets = arrangeAssets(newAssets.toArray());
    }

    public Asset[] getAssets() {
        return this.assets;
    }

    protected int[] countAssets() {
        Iterator iterator = this.CreateIterator();
        int nBonds = 0;
        int nPortfolios = 0;
        while (!iterator.IsDone()) {
            Asset asset = iterator.Next();
            if (asset instanceof Bond) {
                nBonds++;
            } else if (asset instanceof Portfolio) {
                nPortfolios++;
            }
        }
        return new int[]{nPortfolios, nBonds};
    }

    @Override
    public String display() {
        String display = this.name + " [";
        int[] assetsCount = this.countAssets();
        display += "PORT: " + assetsCount[0] + ", BONDS: " + assetsCount[1] + "]";
        return display;
    }
}
```

The children of the `Portfolio` can be other `Portfolio`'s or `Bond`'s (any object of `Asset` class).

To store the children of the `Portfolio`, we use an array like structure. Because we are asked to not use STL or `java.util` structures, we implement our own dynamic array `DynamicAssetArray`. The later has every functionality of a dynamic array and help us to iterate over all the elements.

```java
package lab2.group;

class DynamicAssetArray {
    private Asset[] array;
    private int size = 0;

    public DynamicAssetArray() {
        array = new Asset[10];
    }

    private void ensureCapacity() {
        if (size == array.length) {
            Asset[] newArray = new Asset[array.length * 2];
            for (int i = 0; i < array.length; i++) {
                newArray[i] = array[i];
            }
            array = newArray;
        }
    }

    public void add(Asset asset) {
        ensureCapacity();
        array[size++] = asset;
    }

    public Asset[] toArray() {
        Asset[] copy = new Asset[size];
        for (int i = 0; i < size; i++) {
            copy[i] = array[i];
        }
        return copy;
    }

    public int size() {
        return size;
    }

    public Asset get(int index) {
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
        return array[index];
    }
}
```

Among the methods inside the `Portfolio`, we have the `arrangeAssets`, which arranges `Bond`'s before `Portfolios`. This method is tested inside `PortfolioTest.java`:

```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertSame;


public class PortfolioTest {

    @Test
    public void testAssetsOrder() {
        Portfolio pf1 = new Portfolio(
            "P1",
            new Portfolio(
                "P2",
                new Bond("B1", 100.0, 5, 3, 15, .03),
                new Bond("B1", 100.0, 20, 4, 10, 0.04)
            ),
            new Bond("B1", 500.0, 90, 4, 10, .02),
            new Bond("B1", 300.0, 30, 2, 5, .03)
        );
        pf1.addAsset(
            new Portfolio(
                "P3",
                new Bond("B1", 200, 70, 3, 10, .05)
            )
        );
        pf1.addAsset(new Bond("B1", 250.0, 10, 1, 25, .06));
        Asset[] assets = pf1.getAssets();
        assertEquals(5, assets.length);
        for (int i = 0; i < 3; i++) {
            assertSame(Bond.class, assets[i].getClass());
        }
        for (int i = 3; i < assets.length; i++) {
            assertSame(Portfolio.class, assets[i].getClass());
        }
    }
    
}
```

## Bond
Bond is the Leaf class. It also extends `Asset` and overrides methods in a similar way to the `Portfolio`. Following the TA John suggestion, we also implement the `createIterator` method for the `Bond`: while the `Bond` is a Leaf and has nothing to iterate over, allowing the method to exist and return `null` gives us more flexibility to change the design later on, if necessary.

```java
package lab2.group;

public class Bond extends Asset {
    private String name;
    private double principal;
    private int numberCoupons;
    private int couponFrequency;
    private double couponValue;
    private double yield;

    public Bond(String name, double principal, int numberCoupons, int couponFrequency, double couponValue, double yield) {
        this.name = name;
        this.principal = principal;
        this.numberCoupons = numberCoupons;
        this.couponFrequency = couponFrequency;
        this.couponValue = couponValue;
        this.yield = yield;
    }

    @Override
    public Double calculatePresentValue() {
        int m = this.numberCoupons;
        int k = this.couponFrequency;
        // double y = yield;
        double cc = (this.couponValue * k) / this.principal * 100;

        double c = cc / k / 100;
        double r = 2.5 / k / 100;
        double f = this.principal;
        double t = m;

        double presentValue = c * f * ((1 - Math.pow(1 + r, -t)) / r) + (f / Math.pow(1 + r, t));
        return presentValue;
    }

    @Override
    public Iterator CreateIterator() {
        return null;
    }

    protected double getPrincipal() {
        return this.principal;
    }

    protected double getYield() {
        return this.yield;
    }

    protected int getNumberCoupoms() {
        return this.numberCoupons;
    }

    protected int getCoupomFrequency() {
        return this.couponFrequency;
    }

    protected double getCoupomValue() {
        return this.couponValue;
    }

    @Override
    public DurationResult calculateMacaulayDuration() {
        MacaulayDuration macaulayDuration = new MacaulayDuration(
            this.principal, this.numberCoupons, this.couponFrequency, this.couponValue, this.yield
        );
        double presentValue = this.calculatePresentValue();
        double duration = macaulayDuration.calculateDuration();
        return new DurationResult(presentValue, duration);
    }

    @Override
    public double getMacaulayDuration() {
        DurationResult macaulayDurationResult = this.calculateMacaulayDuration();
        return macaulayDurationResult.duration;
    }
    
    @Override
    public DurationResult calculateModifiedDuration() {
        ModifiedDuration modifiedDuration = new ModifiedDuration(
            this.principal, this.numberCoupons, this.couponFrequency, this.couponValue, this.yield
        );
        double presentValue = this.calculatePresentValue();
        double duration = modifiedDuration.calculateDuration();
        return new DurationResult(presentValue, duration);
    }

    @Override
    public double getModifiedDuration() {
        DurationResult modifiedDurationResult = this.calculateModifiedDuration();
        return modifiedDurationResult.duration;
    }

    @Override
    public String display() {
        String display = "BOND " + this.name + " [";
        display += "PR: " + this.principal + ", ";
        display += "NC: " + this.numberCoupons + ", ";
        display += "F: " + this.couponFrequency + ", ";
        display += "V: " + String.format("%.2f", this.couponValue) + ", ";
        display += "Y: " + String.format("%.2f", this.yield * 100) + "%";
        display += "]";
        return display;
    }

}
```

Inside both Portfolio and Bond, we use the class `DurationResult`. The later class was an alternative to a implementation to the dynamic array from `Java.util`. It is a very simple class that allows us to deal with the calculation of the duration for the Composite and Leaf:

```java
package lab2.group;

class DurationResult {
    double presentValue;
    double duration;

    public DurationResult(double presentValue, double duration) {
        this.presentValue = presentValue;
        this.duration = duration;
    }
}
```

# Strategy
The strategy is composed of 3 classes as well:
1. `Duration`: abstract class with no default methods implemented (if there were default methods implemented, it would be a Template Method).
2. `MacaulayDuration`: class that extends `Duration` and implements the necessary methods.
3. `ModifiedDuration`: class that extends `Duration` and implements the necessary methods in another way.

In our case, the `Duration` has two abstract methods: `calculateDuration` (most important method) and `durationFromBond`, which allows us to instanciate a class in another way.

```java
package lab2.group;

public abstract class Duration {
    protected double principal;
    protected int numberCoupons;
    protected int couponFrequency;
    protected double couponValue;
    protected double yield;

    public Duration(double principal, int numberCoupons, int couponFrequency, double couponValue, double yield) {
        this.principal = principal;
        this.numberCoupons = numberCoupons;
        this.couponFrequency = couponFrequency;
        this.couponValue = couponValue;
        this.yield = yield;

    }
    public abstract double calculateDuration();
    public abstract Duration durationFromBond(Bond bond);
}
```

The Macaulay and Modified durations implement the calculation:

## Macaulay
```java
package lab2.group;

public class MacaulayDuration extends Duration {
    public MacaulayDuration(double principal, int numberCoupons, int couponFrequency, double couponValue, double yield) {
        super(principal, numberCoupons, couponFrequency, couponValue, yield);
    }

    @Override
    public double calculateDuration() {
        int m = this.numberCoupons;
        int k = this.couponFrequency;
        double y = this.yield;
        double cc = (this.couponValue * k) / this.principal * 100;

        double duration = (
            ((1+(y/k)) / (y/k)) - ((100*(1+(y/k)) + (m*((cc/k)-(100*(y/k))))) / ((cc/k)*((Math.pow(1+(y/k), m))-1)+(100*(y/k))))
        ) / k;
        return duration;
    }

    @Override
    public Duration durationFromBond(Bond bond) {
        MacaulayDuration macaulayDuration = new MacaulayDuration(
            bond.getPrincipal(), bond.getNumberCoupoms(), bond.getCoupomFrequency(), bond.getCoupomValue(), bond.getYield()
        );
        return macaulayDuration;    
    }
}
```

## Modified
```java
package lab2.group;

public class ModifiedDuration extends Duration {
    public ModifiedDuration(double principal, int numberCoupons, int couponFrequency, double couponValue, double yield) {
        super(principal, numberCoupons, couponFrequency, couponValue, yield);
    }

    @Override
    public double calculateDuration() {
        int m = this.numberCoupons;
        int k = this.couponFrequency;
        double y = this.yield;
        double cc = (this.couponValue * k) / this.principal * 100;

        double macaulayDuration = (
            ((1+(y/k)) / (y/k)) - ((100*(1+(y/k)) + (m*((cc/k)-(100*(y/k))))) / ((cc/k)*((Math.pow(1+(y/k), m))-1)+(100*(y/k))))
        ) / k;
        double duration = macaulayDuration / (1 + y/k);
        return duration;
    }

    @Override
    public Duration durationFromBond(Bond bond) {
        ModifiedDuration modifiedDuration = new ModifiedDuration(
            bond.getPrincipal(), bond.getNumberCoupoms(), bond.getCoupomFrequency(), bond.getCoupomValue(), bond.getYield()
        );
        return modifiedDuration;
    }
}
```

## Calculation Tests
Later, we use the Excel spreadsheet to validate the results of our classes for both:

### Macaulay Test
```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;


public class MacaulayDurationTest {

    @Test
    public void testChangeYieldCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.02);
        assertEquals(6.619809341, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.06);
        assertEquals(6.182237958, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeCouponFrequencyCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 30, 3, .2 * 100.0 / 3, 0.04);
        assertEquals(6.337957579, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 10, 1, .2 * 100.0 / 1, 0.04);
        assertEquals(6.595206321, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeYearsCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 40, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(10.51462669, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 10, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(3.781333675, mdc3.calculateDuration(), .1);
    }
    
}
```

### Modified Test
```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;


public class ModifiedDurationTest {

    @Test
    public void testChangeYieldCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.02);
        assertEquals(6.554266674, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.06);
        assertEquals(6.002172776, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeCouponFrequencyCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 30, 3, .2 * 100.0 / 3, 0.04);
        assertEquals(6.254563401, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 10, 1, .2 * 100.0 / 1, 0.04);
        assertEquals(6.341544543, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeYearsCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 40, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(10.30845754, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 10, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(3.707189877, mdc3.calculateDuration(), .1);
    }
    
}
```

## Connection with Bond and Portfolio
The strategy classes are later used both inside the `Portfolio` and the `Bond`.

For `Bond` the return is quite simple, and the abstraction is all done by the `Duration`'s classes.

For `Portfolio`, we iterate over the children finding the present value of the bonds and making a weighted average for the duration result.

The method `calculatePresentValue` is used both in `Portfolio` and `Bond` to generate the results desired:

For Portfolio:
```java
@Override
public Double calculatePresentValue() {
    Double presentValue = 0.0;
    for (Asset asset : this.assets) {
        presentValue += asset.calculatePresentValue();
    }
    return presentValue;
}
```

For Bond:
```java
@Override
public Double calculatePresentValue() {
    int m = this.numberCoupons;
    int k = this.couponFrequency;
    // double y = yield;
    double cc = (this.couponValue * k) / this.principal * 100;

    double c = cc / k / 100;
    double r = 2.5 / k / 100;
    double f = this.principal;
    double t = m;

    double presentValue = c * f * ((1 - Math.pow(1 + r, -t)) / r) + (f / Math.pow(1 + r, t));
    return presentValue;
}
```

The present value calculation is tested inside the `PresentValueTest`:

```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;


public class PresentValueTest {

    @Test
    public void testBondPresentValue() {
        double principal1 = 10000.0;
        Bond b1 = new Bond("B1", principal1, 20, 2, principal1 * .2 / 2, .04);
        assertEquals(25399.402, b1.calculatePresentValue(), principal1 * .005);
        double principal2 = 10000.0;
        Bond b2 = new Bond("B1", principal2, 30, 3, principal2 * .2 / 3, .04);
        assertEquals(25427.442, b2.calculatePresentValue(), principal2 * .005);
        double principal3 = 10000.0;
        Bond b3 = new Bond("B1", principal3, 10, 1, principal3 * .2 / 1, .04);
        assertEquals(25316.112, b3.calculatePresentValue(), principal3 * .005);
    }
    
    @Test
    public void testPortfolioPresentValue() {
        double principal1 = 10000.0;
        double principal2 = 10000.0;
        double principal3 = 10000.0;
        Bond b1 = new Bond("B1", principal1, 20, 2, principal1 * .2 / 2, .04);
        Bond b2 = new Bond("B1", principal2, 30, 3, principal2 * .2 / 3, .04);
        Bond b3 = new Bond("B1", principal3, 10, 1, principal3 * .2 / 1, .04);
        Portfolio pf1 = new Portfolio("P1", b1, b2, b3);
        double totalBondMarketValue = b1.calculatePresentValue() + b2.calculatePresentValue() + b3.calculatePresentValue();
        assertEquals(totalBondMarketValue, pf1.calculatePresentValue(), totalBondMarketValue * 0.005);
    }


    @Test
    public void testNestedPortfolioPresentValue() {
        double principal1 = 10000.0;
        double principal2 = 10000.0;
        double principal3 = 10000.0;
        double principal4 = 10000.0;
        Bond b1 = new Bond("B1", principal1, 20, 2, principal1 * .2 / 2, .04);
        Bond b2 = new Bond("B1", principal2, 30, 3, principal2 * .2 / 3, .04);
        Bond b3 = new Bond("B1", principal3, 10, 1, principal3 * .2 / 1, .04);
        Bond b4 = new Bond("B1", principal4, 20, 2, principal4 * .2 / 2, .04);
        Portfolio pf2 = new Portfolio("P1", b3, b4);
        Portfolio pf1 = new Portfolio("P1", b1, b2, pf2);
        double totalBondMarketValue = b1.calculatePresentValue() + b2.calculatePresentValue() + b3.calculatePresentValue() + b4.calculatePresentValue();
        assertEquals(totalBondMarketValue, pf1.calculatePresentValue(), totalBondMarketValue * 0.005);
    }
    
}
```

### Duration in Bond
```java
@Override
public DurationResult calculateMacaulayDuration() {
    MacaulayDuration macaulayDuration = new MacaulayDuration(
        this.principal, this.numberCoupons, this.couponFrequency, this.couponValue, this.yield
    );
    double presentValue = this.calculatePresentValue();
    double duration = macaulayDuration.calculateDuration();
    return new DurationResult(presentValue, duration);
}

@Override
public double getMacaulayDuration() {
    DurationResult macaulayDurationResult = this.calculateMacaulayDuration();
    return macaulayDurationResult.duration;
}

@Override
public DurationResult calculateModifiedDuration() {
    ModifiedDuration modifiedDuration = new ModifiedDuration(
        this.principal, this.numberCoupons, this.couponFrequency, this.couponValue, this.yield
    );
    double presentValue = this.calculatePresentValue();
    double duration = modifiedDuration.calculateDuration();
    return new DurationResult(presentValue, duration);
}

@Override
public double getModifiedDuration() {
    DurationResult modifiedDurationResult = this.calculateModifiedDuration();
    return modifiedDurationResult.duration;
}
```

### Duration in Portfolio
```java
@Override
public DurationResult calculateMacaulayDuration() {
    double summedMarketValue = 0.0;
    double weightedDuration = 0.0;
    for (Asset asset : this.assets) {
        DurationResult result;
        if (asset instanceof Bond) {
            result = ((Bond) asset).calculateMacaulayDuration();
        } else if (asset instanceof Portfolio) {
            result = ((Portfolio) asset).calculateMacaulayDuration();
        } else {
            continue;
        }
        double newMarketValue = result.presentValue;
        double newDuration = result.duration;
        weightedDuration = ((summedMarketValue * weightedDuration) + (newMarketValue * newDuration)) / (summedMarketValue + newMarketValue);
        summedMarketValue += newMarketValue;
    }
    return new DurationResult(summedMarketValue, weightedDuration);
}

@Override
public double getMacaulayDuration() {
    DurationResult macaulayDurationResult = this.calculateMacaulayDuration();
    return macaulayDurationResult.duration;
}

@Override
public DurationResult calculateModifiedDuration() {
    double summedMarketValue = 0.0;
    double weightedDuration = 0.0;
    for (Asset asset : this.assets) {
        DurationResult result;
        if (asset instanceof Bond) {
            result = ((Bond) asset).calculateModifiedDuration();
        } else if (asset instanceof Portfolio) {
            result = ((Portfolio) asset).calculateModifiedDuration();
        } else {
            continue;
        }
        double newMarketValue = result.presentValue;
        double newDuration = result.duration;
        weightedDuration = ((summedMarketValue * weightedDuration) + (newMarketValue * newDuration)) / (summedMarketValue + newMarketValue);
        summedMarketValue += newMarketValue;
    }
    return new DurationResult(summedMarketValue, weightedDuration);
}

@Override
public double getModifiedDuration() {
    DurationResult modifiedDurationResult = this.calculateModifiedDuration();
    return modifiedDurationResult.duration;
}
```

## Duration Test
The results are tested inside `ModifiedDurationTest`:

```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;


public class ModifiedDurationTest {

    @Test
    public void testChangeYieldCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.02);
        assertEquals(6.554266674, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.06);
        assertEquals(6.002172776, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeCouponFrequencyCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 30, 3, .2 * 100.0 / 3, 0.04);
        assertEquals(6.254563401, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 10, 1, .2 * 100.0 / 1, 0.04);
        assertEquals(6.341544543, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeYearsCalculation() {
        ModifiedDuration mdc1 = new ModifiedDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.276789838, mdc1.calculateDuration(), .1);
        ModifiedDuration mdc2 = new ModifiedDuration(100.0, 40, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(10.30845754, mdc2.calculateDuration(), .1);
        ModifiedDuration mdc3 = new ModifiedDuration(100.0, 10, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(3.707189877, mdc3.calculateDuration(), .1);
    }
    
}
```

And inside `MacaulayDurationTest`:

```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertEquals;


public class MacaulayDurationTest {

    @Test
    public void testChangeYieldCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.02);
        assertEquals(6.619809341, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.06);
        assertEquals(6.182237958, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeCouponFrequencyCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 30, 3, .2 * 100.0 / 3, 0.04);
        assertEquals(6.337957579, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 10, 1, .2 * 100.0 / 1, 0.04);
        assertEquals(6.595206321, mdc3.calculateDuration(), .1);
    }

    @Test
    public void testChangeYearsCalculation() {
        MacaulayDuration mdc1 = new MacaulayDuration(100.0, 20, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(6.402325635, mdc1.calculateDuration(), .1);
        MacaulayDuration mdc2 = new MacaulayDuration(100.0, 40, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(10.51462669, mdc2.calculateDuration(), .1);
        MacaulayDuration mdc3 = new MacaulayDuration(100.0, 10, 2, .2 * 100.0 / 2, 0.04);
        assertEquals(3.781333675, mdc3.calculateDuration(), .1);
    }
    
}
```

# Iterator
For the iterator, we have two classes:
1. `AbstractIterator`: which is an abstract class with the basic functionalities of a normal iterator
2. `Iterator`: which implements the Iterator for our specifications of the Composite.

The well known aggregate in our case is the `Asset` class, which has the abstract method `createIterator` later implemented in the derived classes.

```java
package lab2.group;

abstract class AbstractIterator {
    abstract public Asset First();
    abstract public Asset Next();
    abstract public boolean IsDone();
    abstract public int getStep();
    abstract public void setStep(int step);
    abstract public Asset CurrentItem();
}
```

```java
package lab2.group;

public class Iterator extends AbstractIterator {
    private Portfolio portfolio;
    private int current = 0;
    private int step = 1;
    private int nextPortfolioValidIndex = 0;
    private boolean isDone = false;
    private DynamicAssetArray allAssets = new DynamicAssetArray();

    public Iterator(Portfolio portfolio) {
        this.portfolio = portfolio;
        for (Asset asset : portfolio.getAssets()) {
            allAssets.add(asset);
        }
    }

    @Override
    public int getStep() {
        return this.step;
    }

    public void setStep(int step) {
        this.step = step;
    }

    @Override
    public Asset Next() {
        this.current += this.step;
        return CurrentItem();
    }

    @Override
    public boolean IsDone() {
        return isDone;
    }

    @Override
    public Asset CurrentItem() {
        while (allAssets.size() <= this.current && nextPortfolioValidIndex < allAssets.size()) {
            if (allAssets.get(nextPortfolioValidIndex) instanceof Portfolio) {
                Portfolio portfolio = (Portfolio) allAssets.get(nextPortfolioValidIndex);
                for (Asset asset : portfolio.getAssets()) {
                    allAssets.add(asset);
                }
            }
            nextPortfolioValidIndex++;
        }
        if (allAssets.size() > this.current) {
            return allAssets.get(this.current);
        } else {
            isDone = true;
            return null;
        }
    }

    @Override
    public Asset First() {
        this.current = 0;
        isDone = false;
        return allAssets.get(this.current);
    }
}
```

Our iterator follows a similar sequence to a "Tree" data structure implementation: we go over all the elements of one level before going to the next level (with the bonds appearing before the portfolios in each level). The following unittests inside `IteratortTest` help us understand the solution:

```java
package lab2.group;

import org.junit.jupiter.api.Test;
import static org.junit.Assert.assertSame;
import static org.junit.Assert.assertTrue;


public class IteratorTest {

    @Test
    public void testIteratorOrder() {
        Bond b1 = new Bond("B1", 500.0, 90, 4, 10, 0.05);
        Bond b2 = new Bond("B2", 300.0, 30, 2, 5, 0.03);
        Bond b3 = new Bond("B3", 100.0, 5, 3, 15, 0.02);
        Bond b4 = new Bond("B4", 100.0, 20,4, 10, 0.04);
        Bond b5 = new Bond("B5", 250.0, 10, 1, 25, 0.06);
        Bond b6 = new Bond("B6", 750.0, 15, 1, 30, 0.07);

        Portfolio pf4 = new Portfolio("P1", b4, b5, b6);
        Portfolio pf3 = new Portfolio("P2", b3, pf4);
        Portfolio pf2 = new Portfolio("P3", b2, pf3);
        Portfolio pf1 = new Portfolio("P4", b1, pf2);

        Iterator assetIterator = pf1.CreateIterator();
        Asset asset = assetIterator.First();

        assertSame(b1, asset);
        asset = assetIterator.Next();
        assertSame(pf2, asset);
        asset = assetIterator.Next();
        assertSame(b2, asset);
        asset = assetIterator.Next();
        assertSame(pf3, asset);
        asset = assetIterator.Next();
        assertSame(b3, asset);
        asset = assetIterator.Next();
        assertSame(pf4, asset);
        asset = assetIterator.Next();
        assertTrue(asset.equals(b4) || asset.equals(b5) || asset.equals(b6));
        asset = assetIterator.Next();
        assertTrue(asset.equals(b4) || asset.equals(b5) || asset.equals(b6));
        asset = assetIterator.Next();
        assertTrue(asset.equals(b4) || asset.equals(b5) || asset.equals(b6));
        asset = assetIterator.Next();
        assertTrue(asset == null);
        assertTrue(assetIterator.IsDone());
    }
    
}
```


# How to create a similar Project
Change the -DgroupId and -DartifactId to your own values.
```bash
mvn archetype:generate -DgroupId=lab2.group -DartifactId=lab2-fernandourbano-uchicago -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Change the pom to the following changing the groupID and the artifactID to your own values.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>lab2.group</groupId>
  <artifactId>lab2-fernandourbano-uchicago</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.8.2</version>
      <scope>test</scope>
    </dependency>
    <!-- Add more dependencies here -->
  </dependencies>
</project>
```


# Message Queue
- [Message Queue](#lab-3---message-queue)
  - [Set up the lab](#set-up-the-lab)
- [Structure](#structure)
- [Main File](#main-file)
  - [Expected Result for the Main File](#expected-result-for-the-main-file)
- [1. Message Queue](#1-message-queue)
  - [MessageQueue](#messagequeue)
  - [MessageQueueTest](#messagequeuetest)
- [2. Message Types](#2-message-types)
  - [Message](#message)
  - [QueryMsg and ReplyMsg](#querymsg-and-replymsg)
  - [MessageTest](#messagetest)
  - [MsgHeader](#msgheader)
  - [MessageHeaderTest](#messageheadertest)
- [3. Next Message in Message Queue](#3-next-message-in-message-queue)
- [4. Ability to Respond to a Particular QueryMsg with a ReplyMsg](#4-ability-to-respond-to-a-particular-querymsg-with-a-replymsg)
  - [MathProfessor](#mathprofessor)
  - [MathProfessorTest](#mathprofessortest)
  - [MessageQueueOperateTest](#messagequeueoperatetest)
  - [MathStudent](#mathstudent)
  - [MessageQueueOperateWithStudentTest](#messagequeueoperatewithstudenttest)

# Set up the lab
This project was built using Java version 21.0.2. Having it installed, you can run the `App.java` file from the main directory to get the expected results.

This project also uses Apache Maven 3.9.6 to run further unittests in order to check the behaviour of specific classes. Having it installed, run the following commands:

```bash
mvn compile
```

```bash
mvn install
```

While the main file only follows the specifications defined, the unittests go deeper in the implementation. I strongly recommend that you check the unittets as well.

**PLEASE READ THE README AND THE UNITTESTS TO GET A BETTER PERSPECTIVE OF THE IMPLEMENTATION**


# Structure
To properly deliver the homework, a Maven project is used.

- All the classes are built inside the `src/main` folder.
- All the unittests are built inside the `src/test` folder.

To run the unittests, run:
```bash
mvn test
```

Output:

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running lab3.group.MessageQueueOperateWithStudentTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.115 s -- in lab3.group.MessageQueueOperateWithStudentTest
[INFO] Running lab3.group.MessageQueueTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.018 s -- in lab3.group.MessageQueueTest
[INFO] Running lab3.group.MessageQueueOperateTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.011 s -- in lab3.group.MessageQueueOperateTest
[INFO] Running lab3.group.MessageExplainTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.013 s -- in lab3.group.MessageExplainTest
[INFO] Running lab3.group.MessageTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.019 s -- in lab3.group.MessageTest
[INFO] Running lab3.group.MathProfessorTest
[INFO] Tests run: 7, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.081 s -- in lab3.group.MathProfessorTest
[INFO] Running lab3.group.MessageHeaderTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.018 s -- in lab3.group.MessageHeaderTest
[INFO] Running lab3.group.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.014 s -- in lab3.group.AppTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 24, Failures: 0, Errors: 0, Skipped: 0
```


A main file is also built with the specifications provided in the exercise question. It is called `App.java`, also inside `src/main/java/lab3group` folder.

To run `App.java` you should NOT go inside its folder. It should be run from the main folder.

```
.
├── README.md
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── lab3
    │           └── group
    │               ├── App.java
    │               ├── MathProfessor.java
    │               ├── MathStudent.java
    │               ├── Message.java
    │               ├── MessageQueue.java
    │               ├── MsgHeader.java
    │               ├── QueryMsg.java
    │               └── ReplyMsg.java
    └── test
        └── java
            └── lab3
                └── group
                    ├── AppTest.java
                    ├── MathProfessorTest.java
                    ├── MessageExplainTest.java
                    ├── MessageHeaderTest.java
                    ├── MessageQueueOperateTest.java
                    ├── MessageQueueOperateWithStudentTest.java
                    ├── MessageQueueTest.java
                    └── MessageTest.java
```

# Main File
The main file gives a great perspective of what is being done:
```java
public class App  {
    public static void main( String[] args ) {
        System.out.println("\n\nMESSAGE SYSTEM\n");
        System.out.println("1. Instantiate a Message Queue");
        MessageQueue queue = new MessageQueue();

        System.out.println("2. Instanciate a MathStudent");
        MathStudent mathStudent = new MathStudent();

        System.out.println("3. Instanciate a MathProfessor");
        MathProfessor mathProfessor = new MathProfessor();

        System.out.println("4. Instanciate a few query messages");
        Message queryMsg1 = new QueryMsg("1 + 2");
        Message queryMsg2 = new QueryMsg("153 / 9");
        Message queryMsg3 = new QueryMsg("9 * 3");

        System.out.println("5. MathStudent sends the query messages to the queue");
        mathStudent.sendQueryMsg(queryMsg1, queue);
        mathStudent.sendQueryMsg(queryMsg2, queue);
        mathStudent.sendQueryMsg(queryMsg3, queue);

        System.out.println("6. MathProfessor requests the messages and returns the message");
        System.out.println("We expect a reply from the MathProfessor to queryMsg1, answering to the query: 1 + 2:");
        ReplyMsg replyMsg1 = mathProfessor.requestMsg(queue);
        System.out.println("replyMsg1 body: " + replyMsg1.getBody());
        System.out.println("\n");

        System.out.println("We can also take a look at the information of the reply message using three different methods:");
        System.out.println("replyMsg1.explain(): " + replyMsg1.explain());
        System.out.println("replyMsg1.getHeader().display(): " + replyMsg1.getHeader().display());
        System.out.println("replyMsg1.getHeader().toJson(): " + replyMsg1.getHeader().toJson());
        System.out.println("\n");

        System.out.println("7. Math professor requests the next two messages and returns the message to the queue");
        mathProfessor.requestMsgAndReply(queue);
        mathProfessor.requestMsgAndReply(queue);
        System.out.println("\n");
        System.out.println("8. We get the first two messages from the queue and print the body of the messages");
        System.out.println("We expect results to be 17 and 27:");
        System.out.println("queue.getNextFifoMessage().getBody(): " + queue.getNextFifoMessage().getBody());
        System.out.println("queue.getNextFifoMessage().getBody(): " + queue.getNextFifoMessage().getBody());
        System.out.println("\n");

        System.out.println("9. Check that the queue is empty");
        System.out.println("queue.size(): " + queue.size());
        System.out.println("\n");

        System.out.println("10. Send a message as string queries from MathStudent");
        mathStudent.sendQuery("10 + 10", queue);

        System.out.println("11. Set a default queue for MathStudent and send more queries");
        mathStudent.setQueue(queue);
        mathStudent.sendQuery("20 / 2");

        System.out.println("12. Check that the queue has 2 messages");
        System.out.println("queue.size(): " + queue.size());
        System.out.println("\n");

        System.out.println("13. Get the first message, which should be a query message and print the information from it");
        Message msg4 = queue.getNextFifoMessage();
        System.out.println("msg4.getBody(): " + msg4.getBody());
        System.out.println("msg4.explain(): " + msg4.explain());
        System.out.println("msg4.getHeader().display(): " + msg4.getHeader().display());
        System.out.println("\n");

        System.out.println("14. Check that the queue has 1 message");
        System.out.println("queue.size(): " + queue.size());
        System.out.println("\n");

        System.out.println("15. Add an invalid message to the queue");
        mathStudent.sendQuery("When is the test?");
        System.out.println("\n");

        System.out.println("16. mathProfessor responds to the messages");
        mathProfessor.requestMsgAndReply(queue);
        mathProfessor.requestMsgAndReply(queue);
        System.out.println("17. Get the messages from the queue and print them");

        Message msg5 = queue.getNextFifoMessage();
        Message msg6 = queue.getNextFifoMessage();
        System.out.println("We expect the message to be the answer to 20 / 2, which is 10:");
        System.out.println("msg5.getBody(): " + msg5.getBody());
        System.out.println("We expect the message to be the answer to an invalid question:");
        System.out.println("msg6.getBody(): " + msg6.getBody());
        System.out.println("\n");

        System.out.println("18. Send and reply to multiple messages");
        for (int i = 0; i < 10; i++) {
            mathStudent.sendQuery(i + " + " + i, queue);
            System.out.println("Sent " + i + " + " + i + " to message queue");
        }
        for (int i = 0; i < 10; i++) {
            mathProfessor.requestMsgAndReply(queue);
        }
        for (int i = 0; i < 10; i++) {
            Message msg = queue.getNextFifoMessage();
            System.out.println("ReplyMessage body for " + i + " + " + i + ": " + msg.getBody());
        }
    }
}
```

## Expected Result for the Main File
```
MESSAGE SYSTEM

1. Instantiate a Message Queue
2. Instanciate a MathStudent
3. Instanciate a MathProfessor
4. Instanciate a few query messages
5. MathStudent sends the query messages to the queue
6. MathProfessor requests the messages and returns the message
We expect a reply from the MathProfessor to queryMsg1, answering to the query: 1 + 2:
replyMsg1 body: 3


We can also take a look at the information of the reply message using three different methods:
replyMsg1.explain(): ReplyMsg 00000001 to QueryMsg 00000001
replyMsg1.getHeader().display(): MsgHeader{id='00000001', type='REPLY', timestamp='1713151066958', queryHeader=MsgHeader{id='00000001', type='QUERY', timestamp='1713151066955'}}
replyMsg1.getHeader().toJson(): {"id":"00000001","type":"REPLY","timestamp":"1713151066958","queryHeader":{"id":"00000001","type":"QUERY","timestamp":"1713151066955"}}


7. Math professor requests the next two messages and returns the message to the queue


8. We get the first two messages from the queue and print the body of the messages
We expect results to be 17 and 27:
queue.getNextFifoMessage().getBody(): 17
queue.getNextFifoMessage().getBody(): 27


9. Check that the queue is empty
queue.size(): 0


10. Send a message as string queries from MathStudent
11. Set a default queue for MathStudent and send more queries
12. Check that the queue has 2 messages
queue.size(): 2


13. Get the first message, which should be a query message and print the information from it
msg4.getBody(): 10 + 10
msg4.explain(): QueryMsg 00000004
msg4.getHeader().display(): MsgHeader{id='00000004', type='QUERY', timestamp='1713151066963'}


14. Check that the queue has 1 message
queue.size(): 1


15. Add an invalid message to the queue


16. mathProfessor responds to the messages
17. Get the messages from the queue and print them
We expect the message to be the answer to 20 / 2, which is 10:
msg5.getBody(): 10
We expect the message to be the answer to an invalid question:
msg6.getBody(): Invalid query: query must be composed of 2 positive integers separated by an operator (+, -, *, /). Divisions by zero are not allowed. Divisions that generate a rest return only the integer part.


18. Send and reply to multiple messages
Sent 0 + 0 to message queue
Sent 1 + 1 to message queue
Sent 2 + 2 to message queue
Sent 3 + 3 to message queue
Sent 4 + 4 to message queue
Sent 5 + 5 to message queue
Sent 6 + 6 to message queue
Sent 7 + 7 to message queue
Sent 8 + 8 to message queue
Sent 9 + 9 to message queue
ReplyMessage body for 0 + 0: 0
ReplyMessage body for 1 + 1: 2
ReplyMessage body for 2 + 2: 4
ReplyMessage body for 3 + 3: 6
ReplyMessage body for 4 + 4: 8
ReplyMessage body for 5 + 5: 10
ReplyMessage body for 6 + 6: 12
ReplyMessage body for 7 + 7: 14
ReplyMessage body for 8 + 8: 16
ReplyMessage body for 9 + 9: 18
```

# 1. Message Queue
## MessageQueue
Our `MessageQueue` stores all the messages inside the one vector.

```java
public class MessageQueue {
    List<Message> queue = new ArrayList<Message>();

    public MessageQueue() {}

    public Message getNextFifoMessage() {
        if (queue.size() == 0) {
            return null;
        }
        return queue.remove(0);
    }

    public QueryMsg getNextFifoQueryMessage() {
        for (int i = 0; i < queue.size(); i++) {
            if (queue.get(i).getType().equals("QUERY")) {
                return (QueryMsg) queue.remove(i);
            }
        }
        return null;
    }

    public void add(Message msg) {
        queue.add(msg);
    }

    public int size() {
        return queue.size();
    }

    public boolean isEmpty() {
        return queue.isEmpty();
    }
}
```

It has the basic functionalities of a normal queue, like `add` to add a new message, `size`, `isEmpty` and getting the oldest message (of any kind or of a specific class - more explanation in part 3).

## MessageQueueTest
In our `MessageQueueTest` we check if the messages are added correctly and are retreived in the correct order.
```
public class MessageQueueTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }

    @Test
    public void testAddQueryMsgs() {
        Message msg1 = new QueryMsg("Can I have a cookie?");
        Message msg2 = new QueryMsg("Can I have another cookie?");
        Message msg3 = new QueryMsg("Can I have a third cookie?");
        MessageQueue queue = new MessageQueue();
        queue.add(msg1);
        queue.add(msg2);
        queue.add(msg3);
        Message gotMsg1 = queue.getNextFifoMessage();
        assertSame(gotMsg1, msg1);
        Message gotMsg2 = queue.getNextFifoMessage();
        assertSame(gotMsg2, msg2);
        Message gotMsg3 = queue.getNextFifoMessage();
        assertSame(gotMsg3, msg3);
    }

    @Test
    public void testAddMsgs() {
        Message msg1 = new QueryMsg("Can I have a cookie?");
        Message msg2 = new QueryMsg("Can I have another cookie?");
        Message msg3 = new QueryMsg("Can I have a third cookie?");
        Message replyMsg1 = new ReplyMsg(msg1, "Yes!");
        Message replyMsg2 = new ReplyMsg(msg2, "Yes!");
        Message replyMsg3 = new ReplyMsg(msg3, "Last one...");
        MessageQueue queue = new MessageQueue();
        List<Message> msgs = new ArrayList<>();
        queue.add(msg1);
        queue.add(replyMsg1);
        queue.add(msg2);
        queue.add(msg3);
        queue.add(replyMsg2);
        queue.add(replyMsg3);
        msgs.add(msg1);
        msgs.add(replyMsg1);
        msgs.add(msg2);
        msgs.add(msg3);
        msgs.add(replyMsg2);
        msgs.add(replyMsg3);
        for (int i = 0; i < msgs.size(); i++) {
            Message gotMsg = queue.getNextFifoMessage();
            assertSame(gotMsg, msgs.get(i));
        }
        for(int i = 0; i < 3; i++) {
            assertTrue(queue.getNextFifoMessage() == null);
        }
    }
    
}
```

# 2. Message Types
## Message
We have the basic class `Message` which is abstract. It composed of a header and a body.
The header is a `MsgHeader` instance which has metadata about the message and the body has the required information of the class in a string.

```java
public abstract class Message {
    protected MsgHeader header;
    protected String body;

    public MsgHeader getHeader() {
        return header;
    }

    public String getId() {
        return header.getId();
    }

    public String getTimestamp() {
        return header.getTimestamp();
    }

    public String getBody() {
        return body;
    }

    public String getType() {
        return header.getType();
    }

    public abstract String explain();
}
```

## QueryMsg and ReplyMsg
From `Message`, we derive `QueryMsg` and `ReplyMsg`.

Both classes do not require the user to pass the header: the header is created alone based on a class attribute. The class attribute uses an id that increments after every instantiation of the class.

The `ReplyMsg` requires the user to pass the message to which the message is replying or the header of the message. It allows us to track to which message that message is replying.

```java
public class QueryMsg extends Message {
    static int queryClassIdInt = 0;

    public QueryMsg(String body) {
        queryClassIdInt++;
        this.header = new MsgHeader("QUERY", String.format("%08d", queryClassIdInt));
        this.body = body;
    }

    public static void resetCounter() {
        queryClassIdInt = 0;
    }

    @Override
    public String explain() {
        return "QueryMsg " + this.header.getId();
    }
}
```

```java
public class ReplyMsg extends Message {
    static int replyClassIdInt = 0;

    public ReplyMsg(MsgHeader queryHeader, String body) {
        replyClassIdInt++;
        this.header = new MsgHeader("REPLY", String.format("%08d", replyClassIdInt), queryHeader);
        this.body = body;
    }

    public ReplyMsg(Message queryMsg, String body) {
        replyClassIdInt++;
        this.header = new MsgHeader("REPLY", String.format("%08d", replyClassIdInt), queryMsg.header);
        this.body = body;
    }

    public static void resetCounter() {
        replyClassIdInt = 0;
    }

    @Override
    public String explain() {
        return (
            "ReplyMsg "
             + this.header.getId()
             + " to QueryMsg "
             + this.header.getQueryHeader().getId()
        );
    }
}
```

## MessageTest
We test the messages in order to ensure that they are working as expected:

```java
public class MessageTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }
    
    @Test
    public void testQueryMsg() {
        Message msg = new QueryMsg("Can I have a cookie?");
        assertSame("QUERY", msg.getType());
        assertTrue(msg.getId().equals("00000001"));
        assertTrue(msg.getBody().equals("Can I have a cookie?"));
    }

    @Test
    public void testReplyMsg() {
        Message queryMsg = new QueryMsg("Can I have a cookie?");
        Message replyMsg = new ReplyMsg(queryMsg, "Yes!");
        assertTrue(replyMsg.getType().equals("REPLY"));
        assertTrue(replyMsg.getId().equals("00000001"));
        assertTrue(replyMsg.getBody().equals("Yes!"));
        assertTrue(replyMsg.getHeader().getQueryHeaderId().equals(queryMsg.getHeader().getId()));
    }

    @Test
    public void testInstanciateMultipleQueryMsgs() {
        Message queryMsg1 = new QueryMsg("Are we there yet?");
        Message queryMsg2 = new QueryMsg("Are we there?");
        Message queryMsg3 = new QueryMsg("Are we there now?");
        Message queryMsg4 = new QueryMsg("Are we arriving?");
        Message queryMsg5 = new QueryMsg("You said we would be arriving soon!");
        Message queryMsg6 = new QueryMsg("What about now?");
        List<Message> queryMsgs = new ArrayList<>();
        queryMsgs.add(queryMsg1);
        queryMsgs.add(queryMsg2);
        queryMsgs.add(queryMsg3);
        queryMsgs.add(queryMsg4);
        queryMsgs.add(queryMsg5);
        queryMsgs.add(queryMsg6);
        for (int i = 0; i < queryMsgs.size(); i++) {
            assertTrue(queryMsgs.get(i).getType().equals("QUERY"));
            assertTrue(queryMsgs.get(i).getId().equals(String.format("%08d", i + 1)));
        }
    }

    @Test
    public void testInstanciateMultipleReplyMsgs() {
        Message queryMsg1 = new QueryMsg("Are we there yet?");
        Message queryMsg2 = new QueryMsg("Are we there?");
        Message queryMsg3 = new QueryMsg("Are we there now?");
        Message queryMsg4 = new QueryMsg("Are we arriving?");
        Message queryMsg5 = new QueryMsg("You said we would be arriving soon!");
        Message queryMsg6 = new QueryMsg("What about now?");
        Message replyMsg1 = new ReplyMsg(queryMsg1, "Not yet...");
        Message replyMsg2 = new ReplyMsg(queryMsg2, "No...");
        Message replyMsg3 = new ReplyMsg(queryMsg3, "No...");
        Message replyMsg4 = new ReplyMsg(queryMsg4, "No...");
        Message replyMsg5 = new ReplyMsg(queryMsg5, "Not yet...");
        Message replyMsg6 = new ReplyMsg(queryMsg6, "No...");
        List<Message> queryMsgs = new ArrayList<>();
        queryMsgs.add(queryMsg1);
        queryMsgs.add(queryMsg2);
        queryMsgs.add(queryMsg3);
        queryMsgs.add(queryMsg4);
        queryMsgs.add(queryMsg5);
        queryMsgs.add(queryMsg6);
        List<Message> replyMsgs = new ArrayList<>();
        replyMsgs.add(replyMsg1);
        replyMsgs.add(replyMsg2);
        replyMsgs.add(replyMsg3);
        replyMsgs.add(replyMsg4);
        replyMsgs.add(replyMsg5);
        replyMsgs.add(replyMsg6);
        for (int i = 0; i < replyMsgs.size(); i++) {
            assertTrue(queryMsgs.get(i).getType().equals("QUERY"));
            assertTrue(replyMsgs.get(i).getType().equals("REPLY"));
            assertTrue(queryMsgs.get(i).getId().equals(String.format("%08d", i + 1)));
            assertTrue(replyMsgs.get(i).getId().equals(String.format("%08d", i + 1)));
        }
    }
}
```

## MsgHeader
The class message header takes a `type` which is `QUERY` or `REPLY`, an `id`, which is based on the class that creates the message, and possibly a `queryHeader` if that header is of a reply message.

It can be displayed in string format and in json format by calling the desired methods.

String for query message:
```
"MsgHeader{id='00000001', type='QUERY', timestamp='1713130057923'}"
```

Json for reply message:
```
{
    "id": "00000001",
    "type": "REPLY",
    "timestamp": "1713130125533",
    "queryHeader":{
        "id": "00000001",
        "type": "QUERY",
        "timestamp": "1713130125531"
    }
}
```

```java
public class MsgHeader {
    private String id;
    private String type;
    private String timestamp;
    private MsgHeader queryHeader;

    public MsgHeader(String type, String id, MsgHeader queryHeader) {
        this.id = id;
        this.type = type;
        this.timestamp = String.valueOf(System.currentTimeMillis());
        this.queryHeader = queryHeader;
    }

    public MsgHeader(String type, String id) {
        this.id = id;
        this.type = type;
        this.timestamp = String.valueOf(System.currentTimeMillis());
        this.queryHeader = null;
    }

    public String display() {
        String display = (
            "MsgHeader{" +
            "id='" + id + '\'' +
            ", type='" + type + '\'' +
            ", timestamp='" + timestamp + '\''
        );
        if (this.queryHeader != null) {
            display += (
                ", queryHeader=" + queryHeader.display()
            );
        }
        display += "}";
        return display;
    }

    public String toJson() {
        String json = (
            "{" +
            "\"id\":\"" + id + "\"," +
            "\"type\":\"" + type + "\"," +
            "\"timestamp\":\"" + timestamp + "\""
        );
        if (this.queryHeader != null) {
            json += (
                ",\"queryHeader\":" + queryHeader.toJson()
            );
        }
        json += "}";
        return json;
    }

    public String getId() {
        return id;
    }

    public String getType() {
        return type;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public MsgHeader getQueryHeader() {
        return queryHeader;
    }

    public String getQueryHeaderId() {
        if (queryHeader == null) {
            return null;
        }
        return queryHeader.getId();
    }

}
```

## MessageHeaderTest
The following allows us to test that the results of the message header are as expected:
```java
public class MessageHeaderTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }
    

    @Test
    public void testQueryMsgyHeaderToString() {
        MessageQueue queue = new MessageQueue();
        MathStudent mathStudent = new MathStudent();
        mathStudent.setQueue(queue);
        mathStudent.sendQuery("1 + 2");
        Message msg1 = queue.getNextFifoMessage();
        String msg1String = msg1.getHeader().display();
        String expectedMsg1String = (
            "MsgHeader{" +
            "id='00000001'" +
            ", type='QUERY'" +
            ", timestamp="
        );
        assertTrue(msg1String.startsWith(expectedMsg1String));
        mathStudent.sendQuery("2 + 3");
        Message msg2 = queue.getNextFifoMessage();
        String msg2String = msg2.getHeader().display();
        String expectedMsg2String = (
            "MsgHeader{" +
            "id='00000002'" +
            ", type='QUERY'" +
            ", timestamp="
        );
        assertTrue(msg2String.startsWith(expectedMsg2String));
    }

    @Test
    public void testReplyMsgHeaderToString() {
        MessageQueue queue = new MessageQueue();
        MathStudent mathStudent = new MathStudent();
        MathProfessor mathProfessor = new MathProfessor();
        mathStudent.setQueue(queue);
        QueryMsg queryMsg1 = new QueryMsg("1 + 2");
        mathStudent.sendQueryMsg(queryMsg1);
        mathProfessor.requestMsgAndReply(queue);
        Message replyMsg1 = queue.getNextFifoMessage();
        String replyTimestamp = replyMsg1.getTimestamp();
        String queryTimestamp = queryMsg1.getTimestamp();
        String replyMsg1String = replyMsg1.getHeader().display();
        String expectedReplyMsg1String = (
            "MsgHeader{" +
                "id='00000001'" +
                ", type='REPLY'" +
                ", timestamp='" + replyTimestamp + "'" +
                ", queryHeader=" +
                    "MsgHeader{" +
                    "id='00000001'" +
                    ", type='QUERY'" +
                    ", timestamp='" + queryTimestamp + "'" +
                "}" +
            "}"
        );
        assertTrue(replyMsg1String.equals(expectedReplyMsg1String));
    }

    @Test
    public void testQueryMsgHeaderToJson() {
        MessageQueue queue = new MessageQueue();
        MathStudent mathStudent = new MathStudent();
        mathStudent.setQueue(queue);
        mathStudent.sendQuery("1 + 2");
        Message msg1 = queue.getNextFifoMessage();
        String msg1Json = msg1.getHeader().toJson();
        String expectedMsg1Json = (
            "{" +
                "\"id\":\"00000001\"," +
                "\"type\":\"QUERY\"," +
                "\"timestamp\":"
        );
        assertTrue(msg1Json.startsWith(expectedMsg1Json));
        mathStudent.sendQuery("2 + 3");
        Message msg2 = queue.getNextFifoMessage();
        String msg2Json = msg2.getHeader().toJson();
        String expectedMsg2Json = (
            "{" +
                "\"id\":\"00000002\"," +
                "\"type\":\"QUERY\"," +
                "\"timestamp\":"
        );
        assertTrue(msg2Json.startsWith(expectedMsg2Json));
    }

    @Test
    public void testReplyMsgHeaderToJson() {
        MessageQueue queue = new MessageQueue();
        MathStudent mathStudent = new MathStudent();
        MathProfessor mathProfessor = new MathProfessor();
        mathStudent.setQueue(queue);
        QueryMsg queryMsg1 = new QueryMsg("1 + 2");
        mathStudent.sendQueryMsg(queryMsg1);
        mathProfessor.requestMsgAndReply(queue);
        Message replyMsg1 = queue.getNextFifoMessage();
        String replyTimestamp = replyMsg1.getTimestamp();
        String queryTimestamp = queryMsg1.getTimestamp();
        String replyMsg1Json = replyMsg1.getHeader().toJson();
        String expectedReplyMsg1Json = (
            "{" +
                "\"id\":\"00000001\"," +
                "\"type\":\"REPLY\"," +
                "\"timestamp\":\"" + replyTimestamp + "\"," +
                "\"queryHeader\":" +
                    "{" +
                    "\"id\":\"00000001\"," +
                    "\"type\":\"QUERY\"," +
                    "\"timestamp\":\"" + queryTimestamp + "\"" +
                    "}" +
            "}"
        );
        assertTrue(replyMsg1Json.equals(expectedReplyMsg1Json));
    }
}
```

# 3. Next Message in Message Queue
Our Message Queue is built in a way that always the older messages are taken away from the queue first with the `getNextFifoMessage`. When we are dealing with an application that aims to reply to a particular message, we can use the `getNextFifoQueryMessage`. This kind of application only wants to get query messages, to which it can reply.

```java
public Message getNextFifoMessage() {
    if (queue.size() == 0) {
        return null;
    }
    return queue.remove(0);
}

public QueryMsg getNextFifoQueryMessage() {
    for (int i = 0; i < queue.size(); i++) {
        if (queue.get(i).getType().equals("QUERY")) {
            return (QueryMsg) queue.remove(i);
        }
    }
    return null;
}
```

# 4. Ability to Respond to a Particular QueryMsg with a ReplyMsg
## MathProfessor
In order to have the possibility to reply to a particular message, we have an application that reads query messages and creates reply messages to store in the message queue again.

We decide to implement a `MathProfessor` that solves basic mathematical operations.
An instance of the `MathProfessor` can get a message from the message queue and return a reply message with `requestMsg` or already add the reply message to the same queue using `requestMsgAndReply`.

```java
public class MathProfessor {
    
    public MathProfessor() {}

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public int divide(int a, int b) {
        return a / b;
    }

    public Integer operate(String query) {
        String modQuery = query.replaceAll(" ", "");
        String[] queryParts = modQuery.split("\\+|\\-|\\*|\\/");
        if (queryParts.length != 2) {
            return null;
        }
        Pattern pattern = Pattern.compile("\\+|\\-|\\*|\\/");
        Matcher matcher = pattern.matcher(modQuery);
        if (!matcher.find()) {
            return null;
        }
        int a = Integer.parseInt(queryParts[0]);
        String operator = matcher.group();
        int b = Integer.parseInt(queryParts[1]);
        if (operator.equals("+")) {
            return (int) add(a, b);
        } else if (operator.equals("-")) {
            return (int) subtract(a, b);
        } else if (operator.equals("*")) {
            return (int) multiply(a, b);
        } else if (operator.equals("/")) {
            return b == 0 ? null : (int) divide(a, b);
        }
        return null;
    }

    public ReplyMsg requestMsg(MessageQueue msgQueue) {
        QueryMsg queryMsg = msgQueue.getNextFifoQueryMessage();
        if (queryMsg == null) {
            return null;
        }
        String queryBody = queryMsg.getBody();
        Integer result = operate(queryBody);
        if (result == null) {
            ReplyMsg replyMsg = new ReplyMsg(
                queryMsg,
                "Invalid query: query must be composed of 2 positive integers "
                + "separated by an operator (+, -, *, /). Divisions by zero are not allowed. "
                + "Divisions that generate a rest return only the integer part."
            );
            return replyMsg;
        } else {
            ReplyMsg replyMsg = new ReplyMsg(queryMsg, result.toString());
            return replyMsg;
        }
    }

    public void requestMsgAndReply(MessageQueue msgQueue) {
        ReplyMsg replyMsg = requestMsg(msgQueue);
        if (replyMsg != null) {
            msgQueue.add(replyMsg);
        }
    }

    public void run(MessageQueue msgQueue) {
        while (true) {
            ReplyMsg replyMsg = requestMsg(msgQueue);
            if (replyMsg != null) {
                msgQueue.add(replyMsg);
            }
        }
    }
}
```

## MathProfessorTest
We test the `MathProfessor` with `MathProfessorTest`:

```java
public class MathProfessorTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }

    @Test
    public void testCreateAddReplyMsg() {
        Message msg = new QueryMsg("1 + 2");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.equals("3"));
    }

    @Test
    public void testCreateSubstractReplyMsg() {
        Message msg = new QueryMsg("10 - 5");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.equals("5"));
    }

    @Test
    public void testCreateMultiplyReplyMsg() {
        Message msg = new QueryMsg("9 * 3");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.equals("27"));
    }

    @Test
    public void testCreateDivideReplyMsg() {
        Message msg = new QueryMsg("10 / 2");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.equals("5"));
    }

    @Test
    public void testCreateDivisionByZeroReplyMsg() {
        Message msg = new QueryMsg("10 / 0");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.startsWith("Invalid query"));
    }

    @Test
    public void testDivisionWithRestReplyMsg() {
        Message msg = new QueryMsg("48 / 7");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.startsWith("6"));
    }

    @Test
    public void testSubtractionWithNegativeResultReplyMsg() {
        Message msg = new QueryMsg("10 - 15");
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(msg);
        MathProfessor mathProfessor = new MathProfessor();
        ReplyMsg replyMsg = mathProfessor.requestMsg(msgQueue);
        assertTrue(replyMsg.body.startsWith("-5"));
    }
    
}
```

## MessageQueueOperateTest
We test the `MathProfessor` operating with the `MessageQueueOperateTest`:
```java
public class MessageQueueOperateTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }

    @Test
    public void testOperateMessageQueue() {
        Message msg1 = new QueryMsg("1 + 2");
        Message msg2 = new QueryMsg("10 - 5");
        Message msg3 = new QueryMsg("9 * 3");
        MessageQueue queue = new MessageQueue();
        queue.add(msg1);
        queue.add(msg2);
        queue.add(msg3);
        MathProfessor mathProfessor = new MathProfessor();
        mathProfessor.requestMsgAndReply(queue);
        assertSame(queue.size(), 3);
        Message expectedMsg2 = queue.getNextFifoMessage();
        assertSame(expectedMsg2, msg2);
        mathProfessor.requestMsgAndReply(queue);
        assertSame(queue.size(), 2);
        Message expectedMsg1Reply = queue.getNextFifoMessage();
        Message expectedMsg3Reply = queue.getNextFifoMessage();
        assertTrue(expectedMsg1Reply.body.equals("3"));
        assertTrue(expectedMsg3Reply.body.equals("27"));
    }

    @Test
    public void testEveryQueryMsgIsTracebleInQueue() {
        Message msg1 = new QueryMsg("1 + 2");
        Message msg2 = new QueryMsg("10 - 5");
        Message msg3 = new QueryMsg("9 * 3");
        Message msg4 = new QueryMsg("5 + 7");
        Message msg5 = new QueryMsg("15 + 5");
        Message msg6 = new QueryMsg("9 / 3");
        Message msg7 = new QueryMsg("10 * 2");
        Message msg8 = new QueryMsg("14 - 2");
        Message msg9 = new QueryMsg("18 + 15");
        Message msg10 = new QueryMsg("9 / 8");
        MessageQueue queue = new MessageQueue();
        MathProfessor mathProfessor = new MathProfessor();
        queue.add(msg1);
        queue.add(msg2);
        queue.add(msg3);
        mathProfessor.requestMsgAndReply(queue);
        assertSame(3, queue.size());
        queue.add(msg4);
        assertSame(4, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(4, queue.size());
        queue.add(msg5);
        queue.add(msg6);
        assertSame(6, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(6, queue.size());
        queue.add(msg7);
        assertSame(7, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(7, queue.size());
        queue.add(msg8);
        assertSame(8, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(8, queue.size());
        queue.add(msg9);
        queue.add(msg10);
        assertSame(10, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(10, queue.size());
        List<String> messagesIdInQueue = new ArrayList<>();
        int queueSize = 10;
        for (int i = 0; i < queueSize; i++) {
            Message msg = queue.getNextFifoMessage();
            if (msg instanceof QueryMsg) {
                messagesIdInQueue.add(msg.getId());
            } else if (msg instanceof ReplyMsg) {
                messagesIdInQueue.add(((ReplyMsg) msg).getHeader().getQueryHeaderId());
            }
        }
        for (int i = 1; i <= messagesIdInQueue.size(); i++) {
            assertTrue(messagesIdInQueue.contains(String.format("%08d", i)));
        }
    }
    
}
```

## MathStudent
The message student is the application which create query messages.

Each instance of the `MathStudent` can send a query message specying the queue to which it sends the message or to the default queue that can be set with `setQueue`.

```java
public class MathStudent {
    protected MessageQueue queue;

    public MathStudent() {
        this.queue = null;
    }

    public void setQueue(MessageQueue queue) {
        this.queue = queue;
    }

    public void sendQueryMsg(QueryMsg queryMsg) {
        if (this.queue == null) {
            throw new IllegalStateException(
                "Queue has not been set."
                + "Pass a queue to the request or set a default queue."
            );
        }
        this.queue.add(queryMsg);
    }

    public void sendQuery(String query) {
        if (this.queue == null) {
            throw new IllegalStateException(
                "Queue has not been set."
                + "Pass a queue to the request or set a default queue."
            );
        }
        QueryMsg msg = new QueryMsg(query);
        this.sendQueryMsg(msg, this.queue);
    }

    public void sendQueryMsg(QueryMsg queryMsg, MessageQueue queue) {
        queue.add(queryMsg);
    }

    public void sendQuery(String query, MessageQueue queue) {
        QueryMsg msg = new QueryMsg(query);
        this.sendQueryMsg(msg, queue);
    }

}
```

## MessageQueueOperateWithStudentTest
Now we create the same tests using the `MathStudent` as the sender:

```java
public class MessageQueueOperateWithStudentTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }

    @Test
    public void testOperateMessageQueueWithMathStudent() {
        QueryMsg msg1 = new QueryMsg("1 + 2");
        QueryMsg msg2 = new QueryMsg("10 - 5");
        MathStudent mathStudent = new MathStudent();
        MessageQueue queue = new MessageQueue();
        MathProfessor mathProfessor = new MathProfessor();
        mathStudent.sendQueryMsg(msg1, queue);
        mathStudent.sendQueryMsg(msg2, queue);
        mathStudent.setQueue(queue);
        mathStudent.sendQuery("9 * 3");
        mathProfessor.requestMsgAndReply(queue);
        assertSame(queue.size(), 3);
        Message expectedMsg2 = queue.getNextFifoMessage();
        assertSame(expectedMsg2, msg2);
        mathProfessor.requestMsgAndReply(queue);
        assertSame(queue.size(), 2);
        Message expectedMsg1Reply = queue.getNextFifoMessage();
        Message expectedMsg3Reply = queue.getNextFifoMessage();
        assertTrue(expectedMsg1Reply.body.equals("3"));
        assertTrue(expectedMsg3Reply.body.equals("27"));
    }

    @Test
    public void testEveryQueryMsgIsTracebleInQueue() {
        QueryMsg msg1 = new QueryMsg("1 + 2");
        QueryMsg msg2 = new QueryMsg("10 - 5");
        QueryMsg msg3 = new QueryMsg("9 * 3");
        QueryMsg msg4 = new QueryMsg("5 + 7");
        QueryMsg msg5 = new QueryMsg("15 + 5");
        QueryMsg msg6 = new QueryMsg("9 / 3");
        QueryMsg msg7 = new QueryMsg("10 * 2");
        QueryMsg msg8 = new QueryMsg("14 - 2");
        QueryMsg msg9 = new QueryMsg("18 + 15");
        QueryMsg msg10 = new QueryMsg("9 / 8");
        MathStudent mathStudent = new MathStudent();
        MessageQueue queue = new MessageQueue();
        mathStudent.setQueue(queue);
        MathProfessor mathProfessor = new MathProfessor();
        mathStudent.sendQueryMsg(msg1);
        mathStudent.sendQueryMsg(msg2);
        mathStudent.sendQueryMsg(msg3);
        mathProfessor.requestMsgAndReply(queue);
        assertSame(3, queue.size());
        mathStudent.sendQueryMsg(msg4);
        assertSame(4, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(4, queue.size());
        mathStudent.sendQueryMsg(msg5);
        mathStudent.sendQueryMsg(msg6);
        assertSame(6, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(6, queue.size());
        mathStudent.sendQueryMsg(msg7);
        assertSame(7, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(7, queue.size());
        mathStudent.sendQueryMsg(msg8);
        assertSame(8, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(8, queue.size());
        mathStudent.sendQueryMsg(msg9);
        mathStudent.sendQueryMsg(msg10);
        assertSame(10, queue.size());
        mathProfessor.requestMsgAndReply(queue);
        assertSame(10, queue.size());
        List<String> messagesIdInQueue = new ArrayList<>();
        int queueSize = 10;
        for (int i = 0; i < queueSize; i++) {
            Message msg = queue.getNextFifoMessage();
            if (msg instanceof QueryMsg) {
                messagesIdInQueue.add(msg.getId());
            } else if (msg instanceof ReplyMsg) {
                messagesIdInQueue.add(((ReplyMsg) msg).getHeader().getQueryHeaderId());
            }
        }
        for (int i = 1; i <= messagesIdInQueue.size(); i++) {
            assertTrue(messagesIdInQueue.contains(String.format("%08d", i)));
        }
    }
    
}
```

## Explanation
Every time a reply message is replied, the json format of the message header and the string format specify to which message that message is replying. If we want a more direct view, we also have the method `explain` inside `ReplyMsg` and `QueryMsg` that allows us to check again that the result is as expected:

For query message:
```java
@Override
public String explain() {
    return "QueryMsg " + this.header.getId();
}
```

For reply message:
```java
@Override
public String explain() {
    return (
        "ReplyMsg "
            + this.header.getId()
            + " to QueryMsg "
            + this.header.getQueryHeader().getId()
    );
}
```

## MessageExplainTest
The `MessageExplainTest` allows us to test the explanation.

```java
public class MessageExplainTest {

    @BeforeEach
    public void resetMessageCounters() {
        QueryMsg.resetCounter();
        ReplyMsg.resetCounter();
    }

    @Test
    public void testExplainQueryMsg() {
        QueryMsg queryMsg = new QueryMsg("1 + 2");
        String expectedExplain = "QueryMsg 00000001";
        assertTrue(queryMsg.explain().equals(expectedExplain));
    }

    @Test
    public void testExplainReplyMsg() {
        QueryMsg queryMsg1 = new QueryMsg("1 + 2");
        QueryMsg queryMsg2 = new QueryMsg("1 * 2");
        MathProfessor mathProfessor = new MathProfessor();
        MessageQueue msgQueue = new MessageQueue();
        msgQueue.add(queryMsg1);
        msgQueue.add(queryMsg2);
        ReplyMsg replyMsg1 = mathProfessor.requestMsg(msgQueue);
        ReplyMsg replyMsg2 = mathProfessor.requestMsg(msgQueue);
        String expectedExplain1 = "ReplyMsg 00000001 to QueryMsg 00000001";
        String expectedExplain2 = "ReplyMsg 00000002 to QueryMsg 00000002";
        assertTrue(replyMsg1.explain().equals(expectedExplain1));
        assertTrue(replyMsg2.explain().equals(expectedExplain2));
    }

}
```

# Layers, Facade & Strategy
- [Layers, Facade & Strategy](#layers-facade--strategy)
  - [Set up the lab](#set-up-the-lab)
- [Structure](#structure)
- [Main File](#main-file)
  - [Output](#output)
- [Hardware](#hardware)
  - [Hardware Test](#hardware-test)
- [MachineMode](#machinemode)
  - [MachineMode Test](#machinemode-test)
- [AbstractMachineMode and Strategy](#abstractmachinemode-and-strategy)
  - [RampMachineMode](#rampmachinemode)
  - [ConstantCurrentMachineMode](#constantcurrentmachinemode)
  - [ConstantPressureMachineMode](#constantpressuremachinemode)
- [UserInterface](#userinterface)
  - [UserInterface Test](#userinterface-test)
- [How to create a similar Project](#how-to-create-a-similar-project)

# Set up the lab
This project was built using Java version 21.0.2. Having it installed, you can run the `App.java` file from the main directory to get the expected results.

This project also uses Apache Maven 3.9.6 to run further unittests in order to check the behaviour of specific classes. Having it installed, run the following commands:

```bash
mvn compile
```

```bash
mvn install
```

While the main file only follows the specifications defined, the unittests go deeper in the implementation. I strongly recommend that you check the unittets as well.

**PLEASE READ THE README AND THE UNITTESTS TO GET A BETTER PERSPECTIVE OF THE IMPLEMENTATION**

![UML](uml/PhoenixFictitiousManufacturingMachine.jpeg)

# Structure
To properly deliver the homework, a Maven project is used.

- All the classes are built inside the `src/main` folder.
- All the unittests are built inside the `src/test` folder.

To run the unittests, run:
```bash
mvn test
```

Output:
```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running lab4.group.UserInterfaceTest
[INFO] Tests run: 8, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.195 s -- in lab4.group.UserInterfaceTest
[INFO] Running lab4.group.HardwareTest
[INFO] Tests run: 8, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.030 s -- in lab4.group.HardwareTest
[INFO] Running lab4.group.MachineControlTest
[INFO] Tests run: 11, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.063 s -- in lab4.group.MachineControlTest
[INFO] Running lab4.group.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.008 s -- in lab4.group.AppTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 28, Failures: 0, Errors: 0, Skipped: 0
```

A main file is also built with the specifications provided in the exercise question. It is called `App.java`, also inside `src/main/java/lab4/group` folder.

To run `App.java` you should NOT go inside its folder. It should be run from the main folder.

```
.
├── README.md
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   │       └── lab4
│   │           └── group
│   │               ├── AbstractMachineMode.java
│   │               ├── App.java
│   │               ├── ConstantCurrentMachineMode.java
│   │               ├── ConstantPressureMachineMode.java
│   │               ├── Hardware.java
│   │               ├── MachineControl.java
│   │               ├── RampMachineMode.java
│   │               ├── UserInterface.java
│   │               └── manufactured
│   │                   ├── Cwidget.DAS.csv
│   │                   ├── Cwidget.csv
│   │                   ├── Cwidget.reference.csv
│   │                   ├── Manual0000000001.DAS.csv
│   │                   ├── Manual0000000002.DAS.csv
│   │                   ├── Rwidget.DAS.csv
│   │                   ├── Rwidget.csv
│   │                   ├── Rwidget.reference.csv
│   │                   ├── Widget.DAS.csv
│   │                   ├── Widget.csv
│   │                   └── Widget.reference.csv
│   └── test
│       └── java
│           └── lab4
│               └── group
│                   ├── AppTest.java
│                   ├── HardwareTest.java
│                   ├── MachineControlTest.java
│                   ├── UserInterfaceTest.java
│                   └── manufactured
│                       ├── Cwidget.reference.csv
│                       ├── Rwidget.reference.csv
│                       └── Widget.reference.csv
└── uml
    ├── PhoenixFictitiousManufacturingMachine.jpeg
    ├── PhoenixFictitiousManufacturingMachine.puml
    └── classes
        ├── AbstractMachineMode.puml
        ├── ConstantCurrentMachineMode.puml
        ├── ConstantPressureMachineMode.puml
        ├── Hardware.puml
        ├── MachineControl.puml
        ├── RampMachineMode.puml
        └── UserInterface.puml
```

# Main File
Our main file is the `App.java`. It instanciates the `UserInterface` and the required `MachineMode` and `Hardware` layers.


Our implementation is done considering the Facade idea: only the layer above uses the layer below. Furthermore, we try to keep the complexity as lower as possible. With this in mind, the `Hardware` became the most complicated layer. The `MachineMode` is considerably less complicated and allows us to deal only with the main functions of the hardware (inside its own functions).
The `UserInterface` is the most simplified layer: it aims to do the basic proposed and, as an extra, also allows us to set some configurations.

A point of consideration as well is that we created the manual files with numbers: in this way, we can show with more examples the output of the files generated manually.

```java
package lab4.group;

import java.io.File;
import java.util.Map;

public class App 
{
    public static void main( String[] args )
    {

        System.out.println("\nPHOENIX FICTITIOUS MANUFACTURING MACHINE\n");
        Hardware hardware = new Hardware();

        System.out.println("Removing any existing 'csv' files from '/manufactured'...");
        File directory = new File(hardware.getDirectoryName());
        for (File file : directory.listFiles()) {
            if (file.getName().endsWith(".csv") & !file.getName().endsWith("reference.csv")) {
                file.delete();
            }
        }

        Map<String, Integer> currentControlValues;
        MachineControl machineControl = new MachineControl(hardware);
        UserInterface userInterface = new UserInterface(machineControl);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT CONSTANT PRESSURE GOOD PART:\n");
        String constantPressureResult = userInterface.requestRecipe("Widget", "ConstantPressureMachineMode", 50);
        System.out.println("Recipe Result for 'Widget', 'ConstantPressureMachineMode', '50': " + constantPressureResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Widget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT CONSTANT CURRENT GOOD PART:\n");
        String constantCurrentResult = userInterface.requestRecipe("Cwidget", "ConstantCurrentMachineMode", 50);
        System.out.println("Recipe Result for 'Cwidget', 'ConstantCurrentMachineMode', '50': " + constantCurrentResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Cwidget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT RAMP GOOD PART:\n");
        String rampResult = userInterface.requestRecipe("Rwidget", "RampMachineMode", 50);
        System.out.println("Recipe Result for 'Rwidget', 'RampMachineMode', '50': " + rampResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Rwidget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT RAMP EXECUTION ERROR - PARTSIZE BELOW 50:\n");
        String executionErrorRampResult = userInterface.requestRecipe("Rwidget", "RampMachineMode", 30);
        System.out.println("Recipe Result for 'Rwidget', 'RampMachineMode', '30': " + executionErrorRampResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Rwidget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT CONSTANT CURRENT BAD PART - WRONG PART SIZE:\n");
        String badConstantCurrentResult = userInterface.requestRecipe("Cwidget", "ConstantCurrentMachineMode", 70);
        System.out.println("Recipe Result for 'Cwidget', 'ConstantCurrentMachineMode', '70': " + badConstantCurrentResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Cwidget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("IMPLEMENT CONSTANT PRESSURE BAD PART - WRONG PART SIZE:\n");
        String badConstantPressureResult = userInterface.requestRecipe("Widget", "ConstantPressureMachineMode", 80);
        System.out.println("Recipe Result for 'Widget', 'ConstantPressureMachineMode', '70': " + badConstantPressureResult);
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Widget");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("\nIMPLEMENT MANUAL:\n");
        userInterface.setControlValues(11, 60, 120);
        userInterface.manualRun();
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Manual0000000001");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);

        System.out.println("\n" + "=".repeat(75));
        System.out.println("\nIMPLEMENT MANUAL WITH AMPS ABOVE THRESHOLD:\n");
        userInterface.setControlValues(11, 60, 250);
        userInterface.manualRun();
        System.out.println("\nDisplay result:\n");
        userInterface.displayResult("Manual0000000002");
        currentControlValues = userInterface.getControlValues();
        System.out.println("\nCurrent Control Values: " + currentControlValues);
    }
}
```

Output:

```
PHOENIX FICTITIOUS MANUFACTURING MACHINE

Removing any existing 'csv' files from '/manufactured'...

===========================================================================
IMPLEMENT CONSTANT PRESSURE GOOD PART:

Recipe Result for 'Widget', 'ConstantPressureMachineMode', '50': good part

Display result:

Widget,ConstantPressure,50
0,150,0
1,150,2
2,150,4
3,150,6
4,150,8
5,150,10
6,150,12
7,150,14
8,150,16
9,150,18
10,150,20

Current Control Values: {t=10, amps=20, psi=150}

===========================================================================
IMPLEMENT CONSTANT CURRENT GOOD PART:

Recipe Result for 'Cwidget', 'ConstantCurrentMachineMode', '50': good part

Display result:

Cwidget,ConstantCurrent,50
0,50,100
1,48,100
2,46,100
3,44,100
4,42,100
5,40,100
6,38,100
7,36,100
8,34,100
9,32,100
10,30,100
11,28,100
12,26,100
13,24,100
14,22,100
15,20,100
16,18,100
17,16,100
18,14,100
19,12,100
20,10,100

Current Control Values: {t=20, amps=100, psi=10}

===========================================================================
IMPLEMENT RAMP GOOD PART:

Recipe Result for 'Rwidget', 'RampMachineMode', '50': good part

Display result:

Rwidget,Ramp,50
0,0,50
1,10,70
2,20,90
3,30,110
4,40,130
5,50,150
6,60,170
7,70,190
8,80,200
9,90,200
10,100,200
11,100,200
12,100,200
13,100,200
14,100,200
15,100,200
16,100,200
17,100,200
18,100,200
19,100,200
20,100,200
21,100,200
22,100,200
23,100,200
24,100,200
25,100,200
26,100,200
27,100,200
28,100,200
29,100,200
30,100,200

Current Control Values: {t=30, amps=200, psi=100}

===========================================================================
IMPLEMENT RAMP EXECUTION ERROR - PARTSIZE BELOW 50:

Error: Part size must be at least 50 for RampMachineMode
Recipe Result for 'Rwidget', 'RampMachineMode', '30': Execution error

Display result:

Rwidget,Ramp,30

Current Control Values: {t=30, amps=200, psi=100}

===========================================================================
IMPLEMENT CONSTANT CURRENT BAD PART - WRONG PART SIZE:

Recipe Result for 'Cwidget', 'ConstantCurrentMachineMode', '70': bad part

Display result:

Cwidget,ConstantCurrent,70
0,50,120
1,48,120
2,46,120
3,44,120
4,42,120
5,40,120
6,38,120
7,36,120
8,34,120
9,32,120
10,30,120
11,28,120
12,26,120
13,24,120
14,22,120
15,20,120
16,18,120
17,16,120
18,14,120
19,12,120
20,10,120

Current Control Values: {t=20, amps=120, psi=10}

===========================================================================
IMPLEMENT CONSTANT PRESSURE BAD PART - WRONG PART SIZE:

Recipe Result for 'Widget', 'ConstantPressureMachineMode', '70': bad part

Display result:

Widget,ConstantPressure,80
0,180,0
1,180,2
2,180,4
3,180,6
4,180,8
5,180,10
6,180,12
7,180,14
8,180,16
9,180,18
10,180,20

Current Control Values: {t=10, amps=20, psi=180}

===========================================================================

IMPLEMENT MANUAL:


Display result:

Manual
0,60,120
1,60,120
2,60,120
3,60,120
4,60,120
5,60,120
6,60,120
7,60,120
8,60,120
9,60,120
10,60,120

Current Control Values: {t=10, amps=120, psi=60}

===========================================================================

IMPLEMENT MANUAL WITH AMPS ABOVE THRESHOLD:


Display result:

Manual
0,60,200
1,60,200
2,60,200
3,60,200
4,60,200
5,60,200
6,60,200
7,60,200
8,60,200
9,60,200
10,60,200

Current Control Values: {t=10, amps=200, psi=60}
```

# Hardware
Hardware is the lowest layer responsible for actually executing the manufactury process and save the results in the CSV files.
While most functions are not used outside of the hardware itself, we keep them public so we can properly test them.

Furthermore, we made sure to pass the values to the Hardare inside a `Map` steady of directly as parameters. In this way, the `Hardware` becomes more flexible, allowing for additions of new values to be set or removal of current values.

We also make sure to set the minimum and maximum values inside Maps, allowing again for new variables to be added to those configurations as well.

```java
package lab4.group;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

public class Hardware {
    static int numberManual = 1;
    private String filePath = null;
    private int currentTime;
    private String directoryName;
    private Map<String, List<Integer>> history;
    private Map<String, Integer> currentControlVariables;
    private Map<String, Integer> minControlValues;
    private Map<String, Integer> maxControlValues;

    public Hardware() {
        this.minControlValues = Map.of("psi", 0, "amps", 0);
        this.maxControlValues = Map.of("psi", 200, "amps", 200);
        this.currentTime = 0;
        this.history = new HashMap<>();
        this.currentControlVariables = new HashMap<>();
        this.directoryName = "src/main/java/lab4/group/manufactured";
    }

    public Hardware(Map<String, Integer> minControlValues, Map<String, Integer> maxControlValues) {
        this.minControlValues = minControlValues;
        this.maxControlValues = maxControlValues;
        this.currentTime = 0;
        this.history = new HashMap<>();
        this.currentControlVariables = new HashMap<>();
        this.directoryName = "src/main/java/lab4/group/manufactured";
    }

    public void setMinControlValues(Map<String, Integer> minControlValues) {
        this.minControlValues = minControlValues;
    }

    public void setMaxControlValues(Map<String, Integer> maxControlValues) {
        this.maxControlValues = maxControlValues;
    }

    public Map<String, List<Integer>> getHistory() {
        return this.history;
    }

    public void setDirectoryName(String directoryName) {
        this.directoryName = directoryName;
    }

    public String getDirectoryName() {
        return this.directoryName;
    }
    
    public void resetDirectoryName() {
        this.directoryName = "src/main/java/lab4/group/manufactured";
    }

    public String getFilePath() {
        return this.filePath;
    }

    public Map<String, Integer> sendCurrentControlValues() {
        return this.currentControlVariables;
    }

    public void start(String recipeName, String machineMode, int partSize) {
        if (machineMode.endsWith("MachineMode")) {
            machineMode = machineMode.substring(0, machineMode.length() - 11);
        }
        String[] headers = {recipeName, machineMode, String.valueOf(partSize)};
        if (this.filePath != null) {
            throw new IllegalStateException("Hardware is already running.");
        }
        File directory = new File(directoryName);
        this.filePath = new File(directory, recipeName + ".DAS.csv").getPath();
        if (!directory.exists()) {
            directory.mkdirs();
        }
        try (FileWriter writer = new FileWriter(this.filePath)) {
            writer.write(String.join(",", headers) + "\n");
        } catch (IOException e) {
            throw new IllegalStateException("An error occurred while writing " + this.filePath + ": " + e.getMessage() + ".");
        }
    }

    public boolean isRunning() {
        return this.filePath != null;
    }

    public void start() {
        if (this.filePath != null) {
            throw new IllegalStateException("Hardware is already running.");
        }
        File directory = new File(directoryName);
        String manualName = String.format("Manual%010d", numberManual);
        this.filePath = new File(directory, manualName + ".DAS.csv").getPath();
        if (!directory.exists()) {
            directory.mkdirs();
        }
        try (FileWriter writer = new FileWriter(this.filePath)) {
            writer.write("Manual\n");
            numberManual++;
        } catch (IOException e) {
            throw new IllegalStateException("An error occurred while writing " + this.filePath + ": " + e.getMessage() + ".");
        }
    }

    private void writeToFile(String[] data) {
        try (FileWriter writer = new FileWriter(this.filePath, true)) {
            writer.write(String.join(",", data) + "\n");
        } catch (IOException e) {
            throw new IllegalStateException("An error occurred while appending data to " + this.filePath + ": " + e.getMessage() + ".");
        }
    }

    public Map<String, Integer> validateControlValues(Map<String, Integer> controlValues) {
        Map<String, Integer> validatedValues = new HashMap<>();
        for (String key : controlValues.keySet()) {
            int value = controlValues.get(key);
            if (this.minControlValues.containsKey(key)) {
                value = Math.max(value, this.minControlValues.get(key));    
            }
            if (this.maxControlValues.containsKey(key)) {
                value = Math.min(value, this.maxControlValues.get(key));
            }
            validatedValues.put(key, value);
        }
        return validatedValues;
    }

    private void addControlValuesToHistory(Map<String, Integer> controlValues) {
        for (String key : controlValues.keySet()) {
            if (!this.history.containsKey(key)) {
                List<Integer> newHistory = new ArrayList<>();
                newHistory.add(controlValues.get(key));
                this.history.put(key, newHistory);
            } else {
                this.history.get(key).add(controlValues.get(key));
            }
        }
    }

    public void setControlVariables(int deltaTime, Map<String, Integer> controlValues) {
        if (this.filePath == null) {
            throw new IllegalStateException("Hardware is not running.");
        }
        Map<String, Integer> validatedControlValues = validateControlValues(controlValues);
        for (int t = 0; t < deltaTime; t++) {
            currentControlVariables = new HashMap<>(validatedControlValues);
            currentControlVariables.put("t", this.currentTime);
            String[] data = {
                String.valueOf(this.currentTime),
                String.valueOf(validatedControlValues.get("psi")),
                String.valueOf(validatedControlValues.get("amps"))
            };
            this.addControlValuesToHistory(validatedControlValues);
            this.writeToFile(data);
            this.currentTime++;
        }
    }

    public void stop() {
        this.currentTime = 0;
        this.filePath = null;
    }
}
```

## Hardware Test
```java
package lab4.group;

import static org.junit.Assert.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.io.File;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class HardwareTest {
    private Hardware hardware;
    private Map<String, Integer> minControlValues;
    private Map<String, Integer> maxControlValues;

    @BeforeEach
    public void setUp() {
        minControlValues = new HashMap<>();
        maxControlValues = new HashMap<>();
        minControlValues.put("psi", 0);
        maxControlValues.put("psi", 200);
        minControlValues.put("amps", 0);
        maxControlValues.put("amps", 200);

        hardware = new Hardware(minControlValues, maxControlValues);
        hardware.setDirectoryName("src/test/java/lab4/group/manufactured");
    }

    @AfterEach
    public void removeCsvsCreated() {
        File directory = new File(hardware.getDirectoryName());
        for (File file : directory.listFiles()) {
            if (file.getName().endsWith(".csv") & (file.getName().endsWith("reference.csv") == false)) {
                file.delete();
            }
        }
    }

    @Test
    public void testStart() {
        assertNull("Initially filePath should be null", hardware.getFilePath());
        hardware.start("Basic", "Automatic", 5);
        assertNotNull("filePath should not be null after start", hardware.getFilePath());
    }

    @Test
    public void testStartTwice() {
        hardware.start("BasicTest", "Automatic", 5);
        assertThrows(IllegalStateException.class, () -> hardware.start("Basic", "Automatic", 5));
    }

    @Test
    public void testSetControlVariables() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("psi", 50);
        controlValues.put("amps", 150);

        hardware.start("Basic", "Automatic", 5);
        hardware.setControlVariables(1, controlValues);
        assertEquals("Current time should be 0", 0, hardware.sendCurrentControlValues().get("t").intValue());
        assertEquals("psi should be set to 50", 50, hardware.sendCurrentControlValues().get("psi").intValue());
        assertEquals("amps should be set to 150", 150, hardware.sendCurrentControlValues().get("amps").intValue());
    }

    @Test
    public void testBigDeltaTimeControlVariables() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("psi", 50);
        controlValues.put("amps", 150);

        hardware.start("Basic", "Automatic", 5);
        hardware.setControlVariables(10, controlValues);
        assertEquals("Current time should be 9", 9, hardware.sendCurrentControlValues().get("t").intValue());

        hardware.setControlVariables(10, controlValues);
        assertEquals("Current time should be 9", 19, hardware.sendCurrentControlValues().get("t").intValue());
    }

    @Test
    public void testCheckHistoryValues() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("psi", 50);
        controlValues.put("amps", 150);

        hardware.start("Basic", "Automatic", 5);
        hardware.setControlVariables(2, controlValues);
        controlValues.put("psi", 15);
        controlValues.put("amps", 100);
        hardware.setControlVariables(2, controlValues);

        Map<String, List<Integer>> history = hardware.getHistory();
        assertEquals("psi should have 4 values in history", 4, history.get("psi").size());
        assertEquals("amps should have 4 values in history", 4, history.get("amps").size());
        assertEquals("First psi value should be 50", 50, history.get("psi").get(0).intValue());
        assertEquals("Second psi value should be 50", 50, history.get("psi").get(1).intValue());
        assertEquals("Third psi value should be 15", 15, history.get("psi").get(2).intValue());
        assertEquals("Fourth psi value should be 15", 15, history.get("psi").get(3).intValue());
        assertEquals("First amps value should be 150", 150, history.get("amps").get(0).intValue());
        assertEquals("Second amps value should be 150", 150, history.get("amps").get(1).intValue());
        assertEquals("Third amps value should be 100", 100, history.get("amps").get(2).intValue());
        assertEquals("Fourth amps value should be 100", 100, history.get("amps").get(3).intValue());
    }

    @Test
    public void testValidateControlValues() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("psi", -10); // Below the minimum
        controlValues.put("amps", 250); // Above the maximum

        Map<String, Integer> validatedValues = hardware.validateControlValues(controlValues);
        assertEquals("psi should be limited to minimum", 0, validatedValues.get("psi").intValue());
        assertEquals("amps should be limited to maximum", 200, validatedValues.get("amps").intValue());
    }

    @Test
    public void testHistoryValuesValidated() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("psi", -10); // Below the minimum
        controlValues.put("amps", 250); // Above the maximum

        hardware.start("Basic", "Automatic", 5);
        hardware.setControlVariables(2, controlValues);

        Map<String, List<Integer>> history = hardware.getHistory();
        assertEquals("psi should have 2 values in history", 2, history.get("psi").size());
        assertEquals("amps should have 2 values in history", 2, history.get("amps").size());
        assertEquals("First psi value should be 0", 0, history.get("psi").get(0).intValue());
        assertEquals("First amps value should be 200", 200, history.get("amps").get(0).intValue());
    }

    @Test
    public void testStop() {
        hardware.start("Basic", "Automatic", 5);
        hardware.stop();
        assertNull("filePath should be null after stop", hardware.sendCurrentControlValues().get("filePath"));
    }
}
```

# MachineMode
The `MachineMode`is the second layer of the application and serves as a facade for the Hardware. The MachineMode has many functions, but we keep it simple by create functions that wrap up other ones in order to make it simpler. We already have a considerably simpler "UI" than operating direclty the Hardware, which is the most complicated part.

```java
package lab4.group;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Arrays;

public class MachineControl {
    private Hardware hardware;
    private AbstractMachineMode currentMode;
    private Map<String, List<Integer>> expectedHistory;
    private List<String> valuesToCheckInCsv = new ArrayList<>(Arrays.asList("t", "psi", "amps"));

    public MachineControl(Hardware hardware) {
        this.hardware = hardware;
        this.currentMode = null;
        this.expectedHistory = new HashMap<>();
    }

    public void setDirectoryName(String directoryName) {
        this.hardware.setDirectoryName(directoryName);
    }

    public String getDirectoryName() {
        return this.hardware.getDirectoryName();
    }
    
    public void resetDirectoryName() {
        this.hardware.resetDirectoryName();
    }

    public Map<String, List<Integer>> getExpectedHistory() {
        return this.expectedHistory;
    }

    public void setHardwareMinControlValues(Map<String, Integer> minControlValues) {
        this.hardware.setMinControlValues(minControlValues);
    }

    public void setHardwareMaxControlValues(Map<String, Integer> maxControlValues) {
        this.hardware.setMaxControlValues(maxControlValues);
    }

    public Map<String, Integer> getCurrentControlValues() {
        return this.hardware.sendCurrentControlValues();
    }

    public String executeRecipe(String recipeName) {
        File directory = new File(hardware.getDirectoryName());
        String filePath = new File(directory, recipeName + ".csv").getPath();
        String machineMode;
        int partSize;
        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line = br.readLine();
            if (line == null) {
                throw new IllegalStateException("Recipe file is empty");
            }
            String[] values = line.split(",");
            if (values.length != 3) {
                throw new IllegalStateException("Recipe file is not formatted correctly");
            }
            if (!values[0].equals(recipeName)) {
                throw new IllegalStateException("Recipe name does not match");
            }
            machineMode = values[1];
            partSize = Integer.parseInt(values[2]);
        } catch (IOException e) {
            throw new IllegalStateException("An error occurred while reading " + filePath + ": " + e.getMessage() + ".");
        }
        this.startHardware(recipeName, machineMode, partSize);
        this.sendModeInstructions();
        this.stopHardware();
        if (this.checkModeCsvFile(recipeName)) {
            return "good part";
        } else {
            return "bad part";
        }
    }

    public void executeManually(Map<String, Integer> controlValues) {
        this.startHardware();
        this.sendManualInstructions(controlValues);
        this.stopHardware();
    }

    public void startHardware(String recipeName, String machineMode, int partSize) {
        if (partSize < 0 || partSize > 100) {
            throw new IllegalArgumentException("Part size must be between 0 and 100.");
        }
        if (recipeName != null && recipeName.contains(",")) {
            throw new IllegalArgumentException("Recipe name cannot contain a comma and must not be null.");
        }
        expectedHistory.clear();
        hardware.start(recipeName, machineMode, partSize);
        this.setMode(machineMode, partSize);
    }

    public void startHardware() {
        expectedHistory.clear();
        this.currentMode = null;
        hardware.start();
    }

    public void stopHardware() {
        this.currentMode = null;
        hardware.stop();
    }

    public boolean isHardwareRunning() {
        return hardware.isRunning();
    }

    public AbstractMachineMode getCurrentMode() {
        return this.currentMode;
    }

    public String getCurrentModeFullName() {
        if (this.currentMode == null) {
            return null;
        }
        return currentMode.getClass().getName();
    }

    public String getCurrentModeName() {
        String currentModeFullName = this.getCurrentModeFullName();
        if (currentModeFullName == null) {
            return null;
        }
        int lastIndex = currentModeFullName.lastIndexOf('.');
        return currentModeFullName.substring(lastIndex + 1);
    }

    private void setMode(String machineMode, int partSize) {
        try {
            if (machineMode.equals("ConstantCurrentMachineMode")) {
                this.currentMode = new ConstantCurrentMachineMode(partSize);
            } else if (machineMode.equals("ConstantPressureMachineMode")) {
                this.currentMode = new ConstantPressureMachineMode(partSize);
            } else if (machineMode.equals("RampMachineMode")) {
                this.currentMode = new RampMachineMode(partSize);
            } else {
                this.stopHardware();
                throw new IllegalArgumentException("Invalid machine mode");
            }
        } catch (IllegalArgumentException e) {
            this.stopHardware();
            throw e;
        }
    }

    private void addToExpectedHistory(Integer t, Map<String, Integer> controlValues) {
        if (!this.expectedHistory.containsKey("t")) {
            List<Integer> newT = new ArrayList<>();
            this.expectedHistory.put("t", newT);
        }
        this.expectedHistory.get("t").add(t);
        for (String key : controlValues.keySet()) {
            if (!this.expectedHistory.containsKey(key)) {
                List<Integer> newExpectedHistory = new ArrayList<>();
                this.expectedHistory.put(key, newExpectedHistory);
            }
            this.expectedHistory.get(key).add(controlValues.get(key));
        }
    }

    public void sendModeInstructions() {
        if (this.currentMode == null) {
            this.stopHardware();
            throw new IllegalStateException("Machine mode is not set");
        }
        if (!this.hardware.isRunning()) {
            throw new IllegalStateException("Hardware is not running");
        }
        for (int t = 0; t <= this.currentMode.getMaxT(); t++) {
            Map<String, Integer> controlValues = new HashMap<>();
            controlValues.put("amps", this.currentMode.calcAmpsOutput(t));
            controlValues.put("psi", this.currentMode.calcPsiOutput(t));
            Map<String, Integer> copiedControlValues = new HashMap<>(controlValues);
            hardware.setControlVariables(1, controlValues);
            this.addToExpectedHistory(t, copiedControlValues);
        }
    }

    public void sendManualInstructions(Map<String, Integer> controlValues) {
        int T = controlValues.get("T");
        controlValues.remove("T");
        hardware.setControlVariables(T, controlValues);
        this.stopHardware();
    }

    public boolean checkModeCsvFile(String recipeName) {
        File directory = new File(hardware.getDirectoryName());
        String filePath = new File(directory, recipeName + ".DAS.csv").getPath();
        Map<String, List<Integer>> csvHistory = new HashMap<>();
        for (String key : valuesToCheckInCsv) {
            csvHistory.put(key, new ArrayList<>());
        }
        boolean wentOverHeader = false;
        String[] values;
        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line = br.readLine();
            while (true) {
                if (!wentOverHeader) {
                    wentOverHeader = true;
                    line = br.readLine();
                    continue;
                }
                values = line.split(",");
                for (int j = 0; j < values.length; j++) {
                    csvHistory.get(valuesToCheckInCsv.get(j)).add(Integer.parseInt(values[j]));
                }
                line = br.readLine();
                if (line == null) {
                    break;
                }
            }
        } catch (IOException e) {
            this.stopHardware();
            throw new IllegalStateException(
                "An error occurred while reading " + filePath + ": " + e.getMessage() + "."
            );
        }
        directory = new File(hardware.getDirectoryName());
        filePath = new File(directory, recipeName + ".reference.csv").getPath();
        File referenceFile = new File(directory, recipeName + ".reference.csv");
        if (!referenceFile.exists()) {
            this.stopHardware();
            throw new IllegalStateException(
                "Reference file does not exist: " + referenceFile.getPath()
            );
        }
        Map<String, List<Integer>> csvReference = new HashMap<>();
        for (String key : valuesToCheckInCsv) {
            csvReference.put(key, new ArrayList<>());
        }
        wentOverHeader = false;
        String[] referenceValues;
        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line = br.readLine();
            while (true) {
                if (!wentOverHeader) {
                    wentOverHeader = true;
                    line = br.readLine();
                    continue;
                }
                referenceValues = line.split(",");
                for (int j = 0; j < referenceValues.length; j++) {
                    csvReference.get(valuesToCheckInCsv.get(j)).add(Integer.parseInt(referenceValues[j]));
                }
                line = br.readLine();
                if (line == null) {
                    break;
                }
            }
        } catch (IOException e) {
            this.stopHardware();
            throw new IllegalStateException(
                "An error occurred while reading " + filePath + ": " + e.getMessage() + "."
            );
        }
        for (String key : valuesToCheckInCsv) {
            if (!csvReference.get(key).equals(csvHistory.get(key))) {
                return false;
            }
        }
        return true;
    }

    public void displayResult(String fileName) {
        fileName += ".DAS.csv";
        File directory = new File(hardware.getDirectoryName());
        String filePath = new File(directory, fileName).getPath();
        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            String line = br.readLine();
            while (line != null) {
                System.out.println(line);
                line = br.readLine();
            }
        } catch (IOException e) {
            throw new IllegalStateException("An error occurred while reading " + filePath + ": " + e.getMessage() + ".");
        }
    }

}
```

## MachineMode Test
```java
package lab4.group;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.util.Map;
import java.util.Set;
import java.util.HashMap;
import java.util.List;

public class MachineControlTest {
    private MachineControl machineControl;
    private Hardware hardware;
    private Map<String, Integer> minControlValues;
    private Map<String, Integer> maxControlValues;

    @BeforeEach
    public void setUp() {
        minControlValues = new HashMap<>();
        maxControlValues = new HashMap<>();
        minControlValues.put("psi", 0);
        maxControlValues.put("psi", 200);
        minControlValues.put("amps", 0);
        maxControlValues.put("amps", 200);

        hardware = new Hardware(minControlValues, maxControlValues);
        machineControl = new MachineControl(hardware);
        machineControl.setDirectoryName("src/test/java/lab4/group/manufactured");
    }

    @AfterEach
    public void tearDown() {
        File directory = new File(machineControl.getDirectoryName());
        for (File file : directory.listFiles()) {
            if (file.getName().endsWith(".csv") & (file.getName().endsWith("reference.csv") == false)) {
                file.delete();
            }
        }
    }

    @Test
    public void testStartHardware() {
        assertNull(machineControl.getCurrentMode(), "Mode should be null before starting Hardware.");
        machineControl.startHardware("TestRecipe", "ConstantPressureMachineMode", 10);
        assertNotNull(machineControl.getCurrentMode(), "Mode should be set after starting hardware.");
        assertEquals("ConstantPressureMachineMode", machineControl.getCurrentModeName(), "Mode should be set to ConstantPressureMachineMode.");
    }

    @Test
    public void testStopHardware() {
        machineControl.startHardware("TestRecipe", "ConstantPressureMachineMode", 10);
        assertNotNull(machineControl.getCurrentMode(), "Mode should be set after starting hardware.");
        machineControl.stopHardware();
        assertNull(machineControl.getCurrentMode(), "Mode should be null after stopping Hardware.");
    }

    @Test
    public void testExpectedHistoryBehaviourWhenSendingInstructions() {
        assertTrue(machineControl.getExpectedHistory().isEmpty(), "Expected history should be populated after sending instructions.");
        machineControl.startHardware("TestRecipe", "RampMachineMode", 60);
        machineControl.sendModeInstructions();
        assertFalse(machineControl.getExpectedHistory().isEmpty(), "Expected history should be populated after sending instructions.");
    }

    @Test
    public void testExpectedHistoryKeysWhenSendingInstructions() {
        assertTrue(machineControl.getExpectedHistory().isEmpty(), "Expected history should be populated after sending instructions.");
        machineControl.startHardware("TestRecipe", "RampMachineMode", 60);
        machineControl.sendModeInstructions();
        Map<String, List<Integer>> expectedHistory = machineControl.getExpectedHistory();
        Set<String> expectedHistoryKeys = expectedHistory.keySet();
        assertTrue(expectedHistoryKeys.contains("t"), "Expected history should contain t key.");
        assertTrue(expectedHistoryKeys.contains("psi"), "Expected history should contain time key.");
        assertTrue(expectedHistoryKeys.contains("amps"), "Expected history should contain amps key.");
        assertTrue(expectedHistoryKeys.size() == 3, "Expected history should only contain t, psi, and amps keys.");
    }

    @Test
    public void testInvalidMode() {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            machineControl.startHardware("TestRecipe", "InvalidMode", 5);
        });
        assertEquals("Invalid machine mode", exception.getMessage());
    }

    @Test
    public void testInvalidPartSizeForRampMachineMode() {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            machineControl.startHardware("TestRecipe", "RampMachineMode", 5);
        });
        assertEquals("Part size must be at least 50 for RampMachineMode", exception.getMessage());
    }

    @Test
    public void testConstantCurrentCheckModeCsvFile() {
        machineControl.startHardware("Cwidget", "ConstantCurrentMachineMode", 50);
        machineControl.sendModeInstructions();
        assertTrue(machineControl.checkModeCsvFile("Cwidget"), "CSV file should match expected outputs after operations.");
    }

    @Test
    public void testConstantPressureCheckModeCsvFile() {
        machineControl.startHardware("Widget", "ConstantPressureMachineMode", 50);
        machineControl.sendModeInstructions();
        assertTrue(machineControl.checkModeCsvFile("Widget"), "CSV file should match expected outputs after operations.");
    }

    @Test
    public void testRampCheckModeCsvFile() {
        machineControl.startHardware("Rwidget", "RampMachineMode", 50);
        machineControl.sendModeInstructions();
        assertTrue(machineControl.checkModeCsvFile("Rwidget"), "CSV file should match expected outputs after operations.");
    }

    @Test
    public void testRampWithLimitsCheckModeCsvFile() {
        Map<String, Integer> maxControlValues = Map.of("psi", 60, "amps", 60);
        machineControl.setHardwareMaxControlValues(maxControlValues);
        machineControl.startHardware("Rwidget", "RampMachineMode", 50);
        machineControl.sendModeInstructions();
        assertFalse(machineControl.checkModeCsvFile("Rwidget"), "CSV file should match expected outputs after operations.");
    }

    @Test
    public void testManualRun() {
        Map<String, Integer> controlValues = new HashMap<>();
        controlValues.put("T", 10);
        controlValues.put("psi", 60);
        controlValues.put("amps", 60);
        hardware.start();
        machineControl.sendManualInstructions(controlValues);
        File directory = new File(machineControl.getDirectoryName());
        File[] filesArray = directory.listFiles();
        assertNotNull(filesArray, "Directory listFiles returned null.");
        assertTrue(filesArray.length > 0, "No files found in the directory.");
        File latestManualFile = filesArray[filesArray.length - 1];
        for (File file : filesArray) {
            String fileName = file.getName();
            if (fileName.endsWith("DAS.csv") & fileName.startsWith("Manual")) {
                if (file.lastModified() > latestManualFile.lastModified()) {
                    latestManualFile = file;
                }
            }
        }
        try (BufferedReader br = new BufferedReader(new FileReader(latestManualFile))) {
            String line = br.readLine();
            assertTrue(line.equals("Manual"), "First line should be 'Manual'.");
            line = br.readLine();
            Integer currentTime = 0;
            while (line != null) {
                String[] values = line.split(",");
                assertEquals(Integer.toString(currentTime), values[0]);
                assertEquals("60", values[1], "Pressure should be 60.");
                assertEquals("60", values[2], "Amps should be 60.");
                line = br.readLine();
                currentTime++;
            }
        } catch (Exception e) {
            assertTrue(false, "An error occurred while reading the file: " + e.getMessage() + ".");
        }
    }
}
```

# AbstractMachineMode and Strategy
Inside the MachineMode, we have the possibility to use three different algorithms to send the values to the Hardware.

Those are implemented with Strategy pattern using the AbstractMachineMode which has three different subclasses for three different implementations.

```java
package lab4.group;

public abstract class AbstractMachineMode {
    protected static int maxT;
    protected int partSize;

    public abstract int calcAmpsOutput(int T);
    public abstract int getMaxT();

    public abstract int calcPsiOutput(int T);
}
```

They are all used inside the `MachineMode`.


## RampMachineMode
```java
package lab4.group;

public class RampMachineMode extends AbstractMachineMode {
    private static int maxT = 30;

    public RampMachineMode(int partSize) {
        if (partSize < 50) {
            throw new IllegalArgumentException("Part size must be at least 50 for RampMachineMode");
        }
        this.partSize = partSize;
    }

    public int getMaxT() {
        return maxT;
    }

    @Override
    public int calcAmpsOutput(int T) {
        return partSize + T * 20;
    }

    @Override
    public int calcPsiOutput(int T) {
        return Math.min(T * 10, 100);
    }
}
```

## ConstantCurrentMachineMode
```java
package lab4.group;

public class ConstantCurrentMachineMode extends AbstractMachineMode {
    private static int maxT = 20;
    private int partSize;

    public ConstantCurrentMachineMode(int partSize) {
        this.partSize = partSize;
    }

    public int getMaxT() {
        return maxT;
    }

    @Override
    public int calcAmpsOutput(int T) {
        return partSize + 50;
    }

    @Override
    public int calcPsiOutput(int T) {
        return Math.max(50 - T * 2, 10);
    }
}
```

## ConstantPressureMachineMode
```java
package lab4.group;

public class ConstantPressureMachineMode extends AbstractMachineMode {
    private static int maxT = 10;

    public ConstantPressureMachineMode(int partSize) {
        this.partSize = partSize;
    }

    public int getMaxT() {
        return maxT;
    }

    @Override
    public int calcAmpsOutput(int T) {
        return T * 2;
    }

    @Override
    public int calcPsiOutput(int T) {
        return partSize + 100;
    }
}
```

# UserInterface
The `UserInterface` is out last layer. Again, we have multiple functions that are not used everytime, but allow us to proceed with a better implementation, changing settings, modifying the hardware.

In the UserInterface, while keeping the implementation simple, we also try to allow the user to deal with the `MachineMode` and `Hardware` in very personalized ways if wanted.

The main functions to implement the required parts are:
- `setControlValues`: Allow the user to manually set control values in the hardware system.
- `getControlValues`: Allow the user to read the control values.
- `manualRun`: Start the system using the manually controlled values, let it run for T seconds, and then automatically Stop.
- `requestRecipe`: Allow the user to select a recipe that is used to manufacture a particular part and execute that recipe.

It is important to mention that the user never has direct contact with the hardware: that is what makes the layer architecture interesting: the user sends information to the `MachineControl` which talks to the hardware.

One point to consider is that whena recipe is requested, the `MachineControl` validates the output and check if the part was executed correctly.

In order to generate some code that generates "bad part" in the `Hardware` we gave some examples in the tests and in the main file which have a wrong part size specification, so the `MachineControl` returns "bad part".

```java
package lab4.group;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class UserInterface {
    private Map<String, Integer> settedControlValues;
    private MachineControl machineControl;

    public UserInterface(MachineControl machineControl) {
        this.machineControl = machineControl;
        this.settedControlValues = new HashMap<>();
    }

    public void changeMachineControl(MachineControl machineControl) {
        this.machineControl = machineControl;
    }

    public MachineControl getMachineControl() {
        return this.machineControl;
    }

    public void setControlValues(int T, int psi, int amps) {
        this.settedControlValues.put("T", T);
        this.settedControlValues.put("psi", psi);
        this.settedControlValues.put("amps", amps);
    }

    public Map<String, Integer> getControlValues() {
        return this.machineControl.getCurrentControlValues();
    }

    public void manualRun() {
        try {
            this.machineControl.startHardware();
            this.machineControl.sendManualInstructions(settedControlValues);
            this.machineControl.stopHardware();
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    public String requestRecipe(String recipeName, String machineMode, int partSize) {
        String[] headers = {recipeName, machineMode, String.valueOf(partSize)};
        File directory = new File(this.getDirectoryName());
        String filePath = new File(directory, recipeName + ".csv").getPath();
        if (!directory.exists()) {
            directory.mkdirs();
        }
        try (FileWriter writer = new FileWriter(filePath)) {
            writer.write(String.join(",", headers) + "\n");
        } catch (IOException e) {
            System.out.println("An error occurred while writing " + filePath + ": " + e.getMessage() + ".");
        }
        try {
            return this.machineControl.executeRecipe(recipeName);
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
            return "Execution error";
        }
    }

    public void setDirectoryName(String directoryName) {
        this.machineControl.setDirectoryName(directoryName);
    }

    public String getDirectoryName() {
        return this.machineControl.getDirectoryName();
    }
    
    public void resetDirectoryName() {
        this.machineControl.resetDirectoryName();
    }

    public void displayResult(String fileName) {
        try {
            this.machineControl.displayResult(fileName);
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}

```

## UserInterface Test
```java
package lab4.group;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.io.File;
import java.util.HashMap;
import java.util.Map;

public class UserInterfaceTest {
    private UserInterface userInterface;
    private MachineControl machineControl;
    private Hardware hardware;
    private Map<String, Integer> minControlValues;
    private Map<String, Integer> maxControlValues;

    @BeforeEach
    public void setUp() {
        minControlValues = new HashMap<>();
        maxControlValues = new HashMap<>();
        minControlValues.put("psi", 0);
        maxControlValues.put("psi", 200);
        minControlValues.put("amps", 0);
        maxControlValues.put("amps", 200);

        hardware = new Hardware(minControlValues, maxControlValues);
        machineControl = new MachineControl(hardware);
        userInterface = new UserInterface(machineControl);
        machineControl.setDirectoryName("src/test/java/lab4/group/manufactured");
    }

    @AfterEach
    public void removeCsvsCreated() {
        File directory = new File(hardware.getDirectoryName());
        for (File file : directory.listFiles()) {
            if (file.getName().endsWith(".csv") & (file.getName().endsWith("reference.csv") == false)) {
                file.delete();
            }
        }
    }

    @Test
    public void testManualRun() {
        userInterface.setControlValues(10, 60, 120);
        userInterface.manualRun();
        File directory = new File(machineControl.getDirectoryName());
        File[] filesArray = directory.listFiles();
        assertNotNull(filesArray, "Directory listFiles returned null.");
        assertTrue(filesArray.length > 0, "No files found in the directory.");
    }

    @Test
    public void testRequestRecipeGoodPartConstantCurrent() {
        String result = userInterface.requestRecipe("Cwidget", "ConstantCurrentMachineMode", 50);
        assertEquals("good part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeGoodPartConstantPressure() {
        String result = userInterface.requestRecipe("Widget", "ConstantPressureMachineMode", 50);
        assertEquals("good part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeGoodPartRamp() {
        String result = userInterface.requestRecipe("Rwidget", "RampMachineMode", 50);
        assertEquals("good part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeBadPartConstantCurrentWithWrongPartSize() {
        String result = userInterface.requestRecipe("Cwidget", "ConstantCurrentMachineMode", 30);
        assertEquals("bad part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeBadPartConstantPressureWithWrongPartSize() {
        String result = userInterface.requestRecipe("Widget", "ConstantPressureMachineMode", 70);
        assertEquals("bad part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeBadPartRampWithWrongPartSize() {
        String result = userInterface.requestRecipe("Rwidget", "RampMachineMode", 70);
        assertEquals("bad part", result, "The part should be good as per the recipe execution.");
    }

    @Test
    public void testRequestRecipeBadPartWithBiggerHardwareLimitation() {
        machineControl = new MachineControl(hardware);
        machineControl.setHardwareMinControlValues(Map.of("psi", 50, "amps", 50));
        machineControl.setHardwareMaxControlValues(Map.of("psi", 55, "amps", 55));
        userInterface = new UserInterface(machineControl);
        String badResult = userInterface.requestRecipe("Rwidget", "RampMachineMode", 50);
        assertEquals("bad part", badResult, "The part should be bad due to insufficient part size.");
    }
}
```

# How to create a similar Project
Change the -DgroupId and -DartifactId to your own values.
```bash
mvn archetype:generate -DgroupId=lab2.group -DartifactId=lab2-fernandourbano-uchicago -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Change the pom to the following changing the groupID and the artifactID to your own values.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>lab2.group</groupId>
  <artifactId>lab2-fernandourbano-uchicago</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.8.2</version>
      <scope>test</scope>
    </dependency>
    <!-- Add more dependencies here -->
  </dependencies>
</project>
```


# Software Setup and Validation
- [Software Setup and Validation](#lab-5---software-setup-and-validation)
- [Main Structure](#main-structure)
- [Problem 1](#problem-1)
- [Problem 2](#problem-2)

# Main Structure
```
.
├── MPCS-Lab5-Consumer
│   ├── data
│   │   ├── inbox
│   │   │   └── message1.xml
│   │   └── outbox
│   │       ├── Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-02.csv.out
│   │       ├── Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-03.csv.out
│   │       └── [goes on...]
│   ├── pom.xml
│   ├── readme.md
│   ├── src
│   │   └── main
│   │       ├── java
│   │       │   └── camelinaction
│   │       │       └── FileCopierWithCamel.java
│   │       └── resources
│   │           └── log4j.properties
│   └── target
│       ├── chapter1-file-copy-1.0.0.jar
│       ├── classes
│       │   ├── camelinaction
│       │   │   ├── FileCopierWithCamel$1.class
│       │   │   └── FileCopierWithCamel.class
│       │   └── log4j.properties
│       ├── maven-archiver
│       │   └── pom.properties
│       └── maven-status
│           └── maven-compiler-plugin
│               └── compile
│                   └── default-compile
│                       ├── createdFiles.lst
│                       └── inputFiles.lst
├── MPCS-Lab5-Producer
│   ├── data
│   │   ├── inbox
│   │   │   ├── 28-04-24_11-21-02.csv
│   │   │   ├── 28-04-24_11-21-03.csv
│   │   │   └── [goes on...]
│   │   └── outbox
│   ├── pom.xml
│   ├── readme.md
│   ├── src
│   │   └── main
│   │       ├── java
│   │       │   └── camelinaction
│   │       │       └── FileCopierWithCamel.java
│   │       └── resources
│   │           └── log4j.properties
│   └── target
│       ├── chapter1-file-copy-1.0.0.jar
│       ├── classes
│       │   ├── camelinaction
│       │   │   ├── FileCopierWithCamel$1.class
│       │   │   └── FileCopierWithCamel.class
│       │   └── log4j.properties
│       ├── maven-archiver
│       │   └── pom.properties
│       └── maven-status
│           └── maven-compiler-plugin
│               └── compile
│                   └── default-compile
│                       ├── createdFiles.lst
│                       └── inputFiles.lst
├── README.md
├── assets
│   ├── lab5-01.png
│   ├── lab5-02.png
│   ├── lab5-03.png
│   ├── lab5-04.png
│   ├── lab5-05.png
│   ├── lab5-06.png
│   ├── lab5-07.png
│   ├── lab5-08.png
│   └── lab5-09.png
└── chapter1-file-copy
    ├── data
    │   ├── inbox
    │   │   └── message1.xml
    │   └── outbox
    ├── pom.xml
    ├── readme.md
    ├── src
    │   └── main
    │       ├── java
    │       │   └── camelinaction
    │       │       └── FileCopierWithCamel.java
    │       └── resources
    │           └── log4j.properties
    └── target
        ├── classes
        │   ├── camelinaction
        │   │   ├── FileCopierWithCamel$1.class
        │   │   └── FileCopierWithCamel.class
        │   └── log4j.properties
        ├── maven-status
        │   └── maven-compiler-plugin
        │       └── compile
        │           └── default-compile
        │               ├── createdFiles.lst
        │               └── inputFiles.lst
        └── test-classes
```
# Problem 1
In this lab, we first configure Maven, Camel and ActiveMQ and run "chapter1-file-copy".

The configuration required some changes from the original tutorial due to the fact that I am working in a M1 Apple Silicon.

Inside `.m2\repository\org\apache\camel\camel-web\2.0.0\camel-web-2.0.0.pom`:

```
<repositories>
    <repository>
      <id>java.net.m2</id>
      <name>java.net Maven 2 Repo</name>
      <url>http://download.java.net/maven/2</url>
    </repository>
    <repository>
      <id>release.openqa.org</id>
      <name>OpenQA Releases</name>
      <url>http://archiva.openqa.org/repository/releases</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
```

We remove everything after the second `<repository>`:

Removed part:
```
<repository>
    <id>release.openqa.org</id>
    <name>OpenQA Releases</name>
    <url>http://archiva.openqa.org/repository/releases</url>
    <releases>
    <enabled>true</enabled>
    </releases>
    <snapshots>
    <enabled>false</enabled>
    </snapshots>
</repository>
```

Inside the "chapter-file-copy", we also change the repositories part inside the pom:

```pom
  <repositories>
    <repository>
      <id>java.net</id>
      <name>java.net</name>
      <url>http://download.java.net/maven/2</url>
      <layout>default</layout>
    </repository>
    <repository>
      <id>Redhat</id>
      <name>Redhat</name>
      <url>https://maven.repository.redhat.com/ga/</url>
    </repository>
    <repository>
      <id>Repo1</id>
      <name>Repo1</name>
      <url>https://repo1.maven.org/maven2/</url>
    </repository>
  </repositories>
```

and the woodstox:

```pom
<dependency>
    <groupId>org.codehaus.woodstox</groupId>
    <artifactId>wstx-asl</artifactId>
    <version>${woodstox-version}</version>
</dependency>
```

Making this changes, we are able to compile and run the project properly.

![](assets/lab5-01.png)

We tried to execute this multiple times.

# Problem 2
In this problem, we first create the Producer and Consumer projects, which are identical copies of the first project created.

After, we add the assets to the data inbox of the Producer:

<img src="assets/lab5-02.png" alt="Image description" width="150">


Following this, we create a code that follows the step required by the Project:

1.  Consume the input directory "data/input" [CIA v.1: 2.3 & 7.1-7.2; CIA v.2: 2.3 & 6.1-6.2]
2.  Logs the string "RETRIEVED:  ${file:name}" [CIA v.1 & v.2:   Appendix A]
3.  Unmarshals the data read [CIA v.1 & v.2:  3.4]
4.  Runs the CSV translator on the data [CIA v.1 & v.2:  3.4.2]
5.  Splits the body (so that the individual lines go on the queue as individual messages) [CIA v.1 & v.2:  3.4.2]
6.  Sends the output to jms:queue:MPCS_51050_LAB5 [CIA v.1 & v.2: 3.4.2]

The file to do this is inside `MPCS-Lab5-Producer/src/main/java/camelinaction/FileCopierWithCamel.java`:

```java
package camelinaction;

import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

import javax.jms.ConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
    	// create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = 
        	new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent("jms",
            JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
                //from("file:data/inbox?noop=true").to("file:data/outbox");
            	from("file:data/inbox?noop=true")
	            	.log("RETRIEVED:  ${file:name}")
	            	.unmarshal()
	            	.csv()
	            	.split(body())
	            	.to("jms:queue:MPCS_51050_LAB5");
                try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

//                from("file:data/outbox").to("jms:MPCS51050_config_test");
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(10000);

        // stop the CamelContext
        context.stop();
    }
}
```

We can run it by right clicking in the `FileCopieWithCamel.java`:

<img src="assets/lab5-03.png" alt="Image description" width="150">


And using the "Run as", which will generate many log information, such as:

```
[ thread #0 - file://data/inbox] route1                         INFO  RETRIEVED:  28-04-24_11-21-05.csv
[ thread #0 - file://data/inbox] GenericFileConverter           DEBUG Read file data/inbox/28-04-24_11-21-05.csv (no charset)
[ thread #0 - file://data/inbox] SendProcessor                  DEBUG >>>> Endpoint[jms://queue:MPCS_51050_LAB5] Exchange[28-04-24_11-21-05.csv]
[ thread #0 - file://data/inbox] Configuration$CamelJmsTemplate DEBUG Executing callback on JMS Session: ActiveMQSession {id=ID:Fernandos-MacBook-Pro.local-56562-1714406331926-1:100:1,started=false}
[ thread #0 - file://data/inbox] JmsConfiguration               DEBUG Sending JMS message to: queue://MPCS_51050_LAB5 with message: ActiveMQObjectMessage {commandId = 0, responseRequired = false, messageId = null, originalDestination = null, originalTransactionId = null, producerId = null, destination = null, transactionId = null, expiration = 0, timestamp = 0, arrival = 0, brokerInTime = 0, brokerOutTime = 0, correlationId = null, replyTo = null, persistent = true, type = null, priority = 0, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@57000f09, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = {CamelFileLastModified=1714321610370, CamelFileParent=data/inbox, CamelFilePath=data/inbox/28-04-24_11-21-05.csv, CamelFileNameConsumed=28-04-24_11-21-05.csv, breadcrumbId=ID-Fernandos-MacBook-Pro-local-56561-1714406328964-0-298, CamelFileRelativePath=28-04-24_11-21-05.csv, CamelFileLength=25, CamelFileAbsolute=false, CamelFileAbsolutePath=/Users/fernandorochacorreaurbano/mpcs/workspace/MPCS-Lab5-Producer/data/inbox/28-04-24_11-21-05.csv, CamelFileName=28-04-24_11-21-05.csv, CamelFileNameOnly=28-04-24_11-21-05.csv}, readOnlyProperties = false, readOnlyBody = false, droppable = false, jmsXGroupFirstForConsumer = false}
[ thread #0 - file://data/inbox] MulticastProcessor             DEBUG Done sequential processing 1 exchanges
[ thread #0 - file://data/inbox] GenericFileOnCompletion        DEBUG Done processing file: GenericFile[28-04-24_11-21-05.csv] using exchange: Exchange[28-04-24_11-21-05.csv]
[ thread #0 - file://data/inbox] FileUtil                       DEBUG Retrying attempt 0 to delete file: /Users/fernandorochacorreaurbano/mpcs/workspace/MPCS-Lab5-Producer/data/inbox/28-04-24_11-21-05.csv.camelLock
[ thread #0 - file://data/inbox] FileUtil                       DEBUG Tried 1 to delete file: /Users/fernandorochacorreaurbano/mpcs/workspace/MPCS-Lab5-Producer/data/inbox/28-04-24_11-21-05.csv.camelLock with result: true
[ thread #0 - file://data/inbox] FileConsumer                   DEBUG Took 0.015 seconds to poll: data/inbox
```

Running the file will add 100 messages to the queue MPCS_51050_LAB5:

![](assets/lab5-04.png)

Which can be viewed by clicking at the queue:

![](assets/lab5-05.png)

![](assets/lab5-06.png)

Great!

Let's now work with the Consumer. The consumer aims to handle the messages in the queue and transform their format.

For that, we modify the `FileCopierWithCamel.java` inside Consumer to address these changes:

The file is inside `MPCS-Lab5-Consumer/src/main/java/camelinaction`:

```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package camelinaction;

import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

import javax.jms.ConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        // create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = 
        	new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent("jms",
            JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
            	from("jms:queue:MPCS_51050_LAB5")
            	.log("Writing file ${file:name}")
            	.log("RECEIVED:  jms queue: ${body} from file: ${header.CamelFileNameOnly}")
                .convertBodyTo(String.class)
                .to("file:data/outbox?fileName=Thread-${threadName}-${header.CamelFileNameOnly}.out");
               
                try {
    				Thread.sleep(1000);
    			} catch (InterruptedException e) {
    				e.printStackTrace();
    			}

//                from("file:data/outbox").to("jms:MPCS51050_config_test");
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(10000);

        // stop the CamelContext
        context.stop();
    }
}
```

By running the consumer, we get the messages from the queue and add them to the consumer data.

<img src="assets/lab5-07.png" alt="Image description" width="150">

By checking the output:

```
(base) fernandorochacorreaurbano@Fernandos-MacBook-Pro MPCS-Lab5-Consumer % ls data/outbox
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-02.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-53.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-03.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-54.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-04.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-55.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-05.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-56.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-07.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-57.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-08.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-58.csv.out
Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-09.csv.out	Thread-Camel (camel-1) thread #0 - JmsConsumer[MPCS_51050_LAB5]-28-04-24_11-21-59.csv.out

[goes on...]
```

We also check the queue and see that no more messages are in message queue:

![](assets/lab5-08.png)

Attention: we can also add the `noop=true` to the `to` part and the messages would not be removed from the queue. This would be useful to test multiple times if the execution is correct.

```java
.to("file:data/outbox?noop=true&fileName=Thread-${threadName}-${header.CamelFileNameOnly}.out");
```

Checking one file, we see that the output is as requested:

```
[MSFT, 22.81, 118, 22.82, 68]
```

In total, we have 100 files like this:

![](assets/lab5-09.png)


# Producing and Consuming from/to Queue, Publishing and Subscribing to Topic
- [roducing and Consuming from/to Queue, Publishing and Subscribing to Topic](#producing-and-consuming-fromto-queue-publishing-and-subscribing-to-topic)
- [Main Structure](#main-structure)
- [Producer](#producer)
- [Consumer](#consumer)
- [Subscriber](#subscriber)

# Main Structure
```
.
├── MPCS-Lab6-Consumer
│   ├── data
│   │   ├── inbox
│   │   │   └── message1.xml
│   │   └── outbox
│   ├── pom.xml
│   ├── readme.md
│   ├── src
│   │   └── main
│   │       ├── java
│   │       │   └── camelinaction
│   │       │       └── FileCopierWithCamel.java
│   │       └── resources
│   │           └── log4j.properties
│   └── target
│       ├── classes
│       │   ├── camelinaction
│       │   │   ├── FileCopierWithCamel$1.class
│       │   │   └── FileCopierWithCamel.class
│       │   └── log4j.properties
│       └── maven-status
│           └── maven-compiler-plugin
│               └── compile
│                   └── default-compile
│                       ├── createdFiles.lst
│                       └── inputFiles.lst
├── MPCS-Lab6-Producer
│   ├── data
│   │   ├── inbox
│   │   │   ├── 17-05-14_14-24-08.csv
│   │   │   ├── 17-05-14_14-24-09.csv
│   │   │   ├── 17-05-14_14-24-10.csv
│   │   │   ├── 17-05-14_14-24-11.csv
│   │   │   ├── 17-05-14_14-24-12.csv
│   │   │   ├── 17-05-14_14-24-13.csv
│   │   │   ├── 17-05-14_14-24-14.csv
│   │   │   ├── 17-05-14_14-24-15.csv
│   │   │   ├── 17-05-14_14-24-16.csv
│   │   │   ├── 17-05-14_14-24-17.csv
│   │   │   ├── 17-05-14_14-24-26.csv
│   │   │   ├── 17-05-14_14-24-27.csv
│   │   │   ├── 17-05-14_14-24-28.csv
│   │   │   ├── 17-05-14_14-24-29.csv
│   │   │   ├── 17-05-14_14-24-30.csv
│   │   │   ├── 17-05-14_14-24-31.csv
│   │   │   ├── 17-05-14_14-24-32.csv
│   │   │   ├── 17-05-14_14-24-33.csv
│   │   │   ├── 17-05-14_14-24-34.csv
│   │   │   ├── 17-05-14_14-24-35.csv
│   │   │   ├── 17-05-14_14-24-43.csv
│   │   │   ├── 17-05-14_14-24-44.csv
│   │   │   ├── 17-05-14_14-24-45.csv
│   │   │   ├── 17-05-14_14-24-46.csv
│   │   │   ├── 17-05-14_14-24-47.csv
│   │   │   ├── 17-05-14_14-24-48.csv
│   │   │   ├── 17-05-14_14-24-49.csv
│   │   │   ├── 17-05-14_14-24-50.csv
│   │   │   ├── 17-05-14_14-24-51.csv
│   │   │   └── 17-05-14_14-24-52.csv
│   │   └── outbox
│   ├── pom.xml
│   ├── readme.md
│   ├── src
│   │   └── main
│   │       ├── java
│   │       │   └── camelinaction
│   │       │       └── FileCopierWithCamel.java
│   │       └── resources
│   │           └── log4j.properties
│   └── target
│       ├── classes
│       │   ├── camelinaction
│       │   │   ├── FileCopierWithCamel$1$1.class
│       │   │   ├── FileCopierWithCamel$1.class
│       │   │   └── FileCopierWithCamel.class
│       │   └── log4j.properties
│       └── maven-status
│           └── maven-compiler-plugin
│               └── compile
│                   └── default-compile
│                       ├── createdFiles.lst
│                       └── inputFiles.lst
└── MPCS-Lab6-Subscriber
    ├── data
    │   ├── inbox
    │   │   └── message1.xml
    │   └── outbox
    ├── pom.xml
    ├── readme.md
    ├── src
    │   └── main
    │       ├── java
    │       │   └── camelinaction
    │       │       └── FileCopierWithCamel.java
    │       └── resources
    │           └── log4j.properties
    └── target
        ├── classes
        │   ├── camelinaction
        │   │   ├── FileCopierWithCamel$1$1.class
        │   │   ├── FileCopierWithCamel$1$2.class
        │   │   ├── FileCopierWithCamel$1$3.class
        │   │   ├── FileCopierWithCamel$1.class
        │   │   └── FileCopierWithCamel.class
        │   └── log4j.properties
        └── maven-status
            └── maven-compiler-plugin
                └── compile
                    └── default-compile
                        ├── createdFiles.lst
                        └── inputFiles.lst
```
It is important before running the files to ensure that we have the right version of Java.

First, run the following commands separetely in your terminal:

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home
```

```
export PATH=$JAVA_HOME/bin:$PATH
```

```
java -version
```

The output should be:
```
java version "1.8.0_411"
Java(TM) SE Runtime Environment (build 1.8.0_411-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.411-b09, mixed mode)
```

After changing to the right version of Java, inside each of the applications clean and compile the projects:

```
mvn install clean 
```

```
mvn clean compile
```


# Producer
The producer contains inside the `data/inbox` the csv files that will be sent the the `queue:MPCS_51050_LAB6`:

```
(base) fernandorochacorreaurbano@Fernandos-MacBook-Pro inbox % ls
17-05-14_14-24-08.csv	17-05-14_14-24-11.csv	17-05-14_14-24-14.csv	17-05-14_14-24-17.csv	17-05-14_14-24-28.csv	17-05-14_14-24-31.csv	17-05-14_14-24-34.csv	17-05-14_14-24-44.csv	17-05-14_14-24-47.csv	17-05-14_14-24-50.csv
17-05-14_14-24-09.csv	17-05-14_14-24-12.csv	17-05-14_14-24-15.csv	17-05-14_14-24-26.csv	17-05-14_14-24-29.csv	17-05-14_14-24-32.csv	17-05-14_14-24-35.csv	17-05-14_14-24-45.csv	17-05-14_14-24-48.csv	17-05-14_14-24-51.csv
17-05-14_14-24-10.csv	17-05-14_14-24-13.csv	17-05-14_14-24-16.csv	17-05-14_14-24-27.csv	17-05-14_14-24-30.csv	17-05-14_14-24-33.csv	17-05-14_14-24-43.csv	17-05-14_14-24-46.csv	17-05-14_14-24-49.csv	17-05-14_14-24-52.csv
```

The data is sent from the data/inbox to the queue specified.

Because the `noop=true`, the files are also kept in the data inbox when they are sent to the queue. This allows us to more easily debug the application.

```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package camelinaction;
import org.apache.camel.CamelContext;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;

import javax.jms.ConnectionFactory;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;
import org.apache.camel.impl.DefaultCamelContext;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        // create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = 
        	new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent("jms",
            JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
            	from("file:data/inbox?noop=true")
            	.log("RETRIEVED:  ${file:name}")
            	.unmarshal().csv().split(body())
            	.process(new Processor() {
                    public void process(Exchange e) throws Exception {
                    	System.out.println("MESSAGE FROM FILE: " + e.getIn().getHeader("CamelFileName") + 
                    		" is heading to MPCS_51050_Lab6 Queue for Stock: " +  (e.getIn().getBody(String.class).split("\t"))[0].substring(1));
                    }
            	}).to("jms:queue:MPCS_51050_LAB6");
                try {
			        Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                // from("file:data/outbox").to("jms:MPCS51050_config_test");
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(2000);

        // stop the CamelContext
        context.stop();
    }
}
```

Once the application is runned, the files go to the queue:

![](camel-producer-subscriber/assets/lab6-01.png)

In terminal, inside the producer app run the following to execute the producer:

```
mvn install clean 
```

```
mvn clean compile
```

```
mvn exec:java -Dexec.mainClass=camelinaction.FileCopierWithCamel
```

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/fernandorochacorreaurbano/.m2/repository/org/slf4j/slf4j-log4j12/1.7.5/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/fernandorochacorreaurbano/.m2/repository/org/apache/activemq/activemq-all/5.9.0/activemq-all-5.9.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
[ion.FileCopierWithCamel.main()] DefaultCamelContext            DEBUG Adding routes from 
continues...
```

Your answer should be like this (the extra 30 in pending messages)

![](camel-producer-subscriber/assets/lab6-02.png)

Each part of the code is specific to proceed with a certain instruction.

1. Consumes the input directory "data/input" [CIA 2.3 & 7.1-7.2]

```
from("file:data/input?noop=true")
```

2. Logs a string of this format: "RECEIVED: ${file:name}" [CIA Appendix A]
```
.log("RECEIVED: ${file:name}")
```

3 & 4. Unmarshals the data read & Runs the CSV translator on the data [CIA 3.4 & 3.4.2]
```
.unmarshal().csv()
```

5. Splits the body (so that the individual lines go on the queue as individual messages) [CIA 3.4.2]
```
.split(body())
```

6. Creates a new Processor() that prints out the header's CamelFileName along with the stock in the file [CIA 2.5.2]

```
.process(new Processor() {
    public void process(Exchange e) throws Exception {
        System.out.println("MESSAGE FROM FILE: " + e.getIn().getHeader("CamelFileName") + 
            " is heading to MPCS_51050_LAB6 Queue for Stock: " +  (e.getIn().getBody(String.class).split("\t"))[0].substring(1));
    }
})
```

7. Sends the output to the destination endpoint: jms:queue:MPCS_51050_LAB6 [CIA 3.4.2]
```
.to("jms:queue:MPCS_51050_LAB6")
```

The following is an example of message:

![](camel-producer-subscriber/assets/lab6-03.png)

# Consumer
```java
package camelinaction;

import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

import javax.jms.ConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        // create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = 
        	new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent("jms",
            JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
            	from("jms:queue:MPCS_51050_LAB6").log("RECEIVED:  jms queue: ${body} from file: ${header.CamelFileNameOnly}")
                .choice()
                	.when(body().regex(".*MSFT.*"))
                		.to("jms:topic:MPCS_51050_LAB6_TOPIC_MSFT")
            		.when(body().regex(".*ORCL.*"))
            			.to("jms:topic:MPCS_51050_LAB6_TOPIC_ORCL")
            		.when(body().regex(".*IBM.*"))
            			.to("jms:topic:MPCS_51050_LAB6_TOPIC_IBM");
            	
	            try {
                	Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
                //from("file:data/outbox").to("jms:MPCS51050_config_test");
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(10000);

        // stop the CamelContext
        context.stop();
    }
}
```

Each of the parts is responsible for executing one of the instructions:

1. Consume all 30 messages from the MPCS_51050_LAB6 queue [CIA 7.3]

```
from("jms:queue:MPCS_51050_LAB6")
```

2. Log the string "RECEIVED: jms queue: ${body} from file: ${header.CamelFileNameOnly}" [CIA Appendix A]

```
.log("RECEIVED: jms queue: ${body} from file: ${header.CamelFileNameOnly}")
```

3 & 4. Create a Content-Based Router [CIA 2.5, especially 2.5.1]

```
.choice()
    .when(body().regex(".*MSFT.*"))
        .to("jms:topic:MPCS_51050_LAB6_TOPIC_MSFT")
    .when(body().regex(".*ORCL.*"))
        .to("jms:topic:MPCS_51050_LAB6_TOPIC_ORCL")
    .when(body().regex(".*IBM.*"))
        .to("jms:topic:MPCS_51050_LAB6_TOPIC_IBM");
```


To execute the Consumer app, in terminal, run:

```
mvn install clean 
```

```
mvn clean compile
```

```
mvn exec:java -Dexec.mainClass=camelinaction.FileCopierWithCamel
```

The result sends the messages to the topics.

![](camel-producer-subscriber/assets/lab6-04.png)


# Subscriber 
```java
package camelinaction;

import org.apache.camel.CamelContext;

import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.impl.DefaultCamelContext;

import javax.jms.ConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        // create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = 
        	new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent("jms",
            JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
            	
            from("jms:topic:MPCS_51050_LAB6_TOPIC_MSFT")
            .log("SUBSCRIBER RECEIVED: jms MSFT queue: ${body} from file: ${header.CamelFileNameOnly}")
            .process(new Processor() {
                public void process(Exchange e) throws Exception {
                    String[] array = e.getIn().getBody(String.class).split("\t");
                    StringBuilder sb = new StringBuilder();
                    sb.append("Stock:"+array[0].substring(1)+"|");
                    sb.append("BidPrice: "+array[1]+"|");
                    sb.append("BidQuantity: "+array[2]+"|");
                    sb.append("AskPrice: "+array[3]+"|");
                    sb.append("AskQuantity: "+array[4]);
                    e.getIn().setBody(sb.toString());
                }
            }).to("jms:queue:MPCS_51050_LAB6_ALL");

            from("jms:topic:MPCS_51050_LAB6_TOPIC_ORCL")
            .log("SUBSCRIBER RECEIVED: jms ORCL queue: ${body} from file: ${header.CamelFileNameOnly}")
            .process(new Processor() {
                public void process(Exchange e) throws Exception {
                    String[] array = e.getIn().getBody(String.class).split("\t");
                    StringBuilder sb = new StringBuilder();
                    sb.append("Stock: "+array[0].substring(1)+"|");
                    sb.append("BidPrice: "+array[1]+"|");
                    sb.append("BidQuantity: "+array[2]+"|");
                    sb.append("AskPrice: "+array[3]+"|");
                    sb.append("AskQuantity: "+array[4]);
                    e.getIn().setBody(sb.toString());
                }
            }).to("jms:queue:MPCS_51050_LAB6_ALL");

            from("jms:topic:MPCS_51050_LAB6_TOPIC_IBM")
            .log("SUBSCRIBER RECEIVED: jms IBM queue: ${body} from file: ${header.CamelFileNameOnly}")
            .process(new Processor() {
                public void process(Exchange e) throws Exception {
                    String[] array = e.getIn().getBody(String.class).split("\t");
                    StringBuilder sb = new StringBuilder();
                    sb.append("Stock:"+array[0].substring(1)+"|");
                    sb.append("BidPrice: "+array[1]+"|");
                    sb.append("BidQuantity: "+array[2]+"|");
                    sb.append("AskPrice: "+array[3]+"|");
                    sb.append("AskQuantity: "+array[4]);
                    e.getIn().setBody(sb.toString());
                }
            }).to("jms:queue:MPCS_51050_LAB6_ALL");
            try {
            	Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

//                from("file:data/outbox").to("jms:MPCS51050_config_test");
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(15000);

        // stop the CamelContext
        context.stop();
    }
}
```

Each of the following is responsible for prodecing with one of the instructions:

1. Read from each of the Topic queues created by the Consumer
```
from("jms:topic:MPCS_51050_LAB6_TOPIC_MSFT")
from("jms:topic:MPCS_51050_LAB6_TOPIC_ORCL")
from("jms:topic:MPCS_51050_LAB6_TOPIC_IBM")
```

2. Log Activity
```
.log("SUBSCRIBER RECEIVED: jms MSFT queue: ${body} from file: ${header.CamelFileNameOnly}")
.log("SUBSCRIBER RECEIVED: jms ORCL queue: ${body} from file: ${header.CamelFileNameOnly}")
.log("SUBSCRIBER RECEIVED: jms IBM queue: ${body} from file: ${header.CamelFileNameOnly}")
```

3 & 4: Extract data from the message and identify stock details
```
String[] array = e.getIn().getBody(String.class).split("\t");
```

5 & 6. Format the message and modify the exchange
```
StringBuilder sb = new StringBuilder();
sb.append("Stock:"+array[0].substring(1)+"|");
sb.append("BidPrice: "+array[1]+"|");
sb.append("BidQuantity: "+array[2]+"|");
sb.append("AskPrice: "+array[3]+"|");
sb.append("AskQuantity: "+array[4]);
e.getIn().setBody(sb.toString());
```

In order to properly run this file, we must run the producer and the consumer together as well.

For that, organize three terminals side by side. In each of those, "cd" yourself into one of the three projects.

In each of those make sure you run:

```
mvn install clean 
```

```
mvn clean compile
```

Afterwards, run the same code inside each:

```
mvn exec:java -Dexec.mainClass=camelinaction.FileCopierWithCamel
```

The code is the same because we kept the name of the file.

The order in which you run is important. Start from Producer, run Subscriber and than run Consumer.

![](camel-producer-subscriber/assets/lab6-05.png)

![](camel-producer-subscriber/assets/lab6-06.png)

![](camel-producer-subscriber/assets/lab6-07.png)

The result is can be viewed at the ActiveMQ:

![](camel-producer-subscriber/assets/lab6-08.png)

Now, all the messages are in the MPCS51050_LAB6_ALL as expected!


# Producing Messages from RSS Feeds to Queue leveraging xPath and Regex
- [Producing Messages from RSS Feeds to Queue leveraging xPath & Regex](#producing-messages-from-rss-feeds-to-queue-leveranging-xpath-and-regex)
- [Main Structure](#main-structure)
- [Producer](#producer)

# Main Structure
```
.
├── data
│   ├── inbox
│   │   └── message1.xml
│   └── outbox
│       ├── ID-Fernandos-MacBook-Pro-local-50900-1715037911339-0-16
│       ├── ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-21
│       ├── ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-25
│       ├── ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-30
│       └── [messages continue...]
├── logs
│   └── app.log
├── pom.xml
├── readme.md
├── src
│   └── main
│       ├── java
│       │   └── camelinaction
│       │       └── FileCopierWithCamel.java
│       └── resources
│           └── log4j.properties
└── target
    ├── classes
    │   ├── camelinaction
    │   │   ├── FileCopierWithCamel$1$1.class
    │   │   ├── FileCopierWithCamel$1.class
    │   │   └── FileCopierWithCamel.class
    │   └── log4j.properties
    └── maven-status
        └── maven-compiler-plugin
            └── compile
                └── default-compile
                    ├── createdFiles.lst
                    └── inputFiles.lst
```

It is important before running the file to ensure that we have the right version of Java.

First, run the following commands separetely in your terminal:

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-1.8.jdk/Contents/Home
```

```
export PATH=$JAVA_HOME/bin:$PATH
```

```
java -version
```

The output should be:
```
java version "1.8.0_411"
Java(TM) SE Runtime Environment (build 1.8.0_411-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.411-b09, mixed mode)
```

After changing to the right version of Java, inside each of the applications clean and compile the projects:

```
mvn install clean 
```

```
mvn clean compile
```

# Producer
```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package camelinaction;
import org.apache.camel.CamelContext;
import org.apache.camel.Exchange;
import org.apache.camel.LoggingLevel;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

import java.util.Arrays;
import java.util.List;

import javax.jms.ConnectionFactory;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.component.jms.JmsComponent;



public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        // create CamelContext
        CamelContext context = new DefaultCamelContext();

        // connect to ActiveMQ JMS broker listening on localhost on port 61616
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
        context.addComponent(
        		"jms", JmsComponent.jmsComponentAutoAcknowledge(connectionFactory)
        );
        
        // add our route to the CamelContext
        context.addRoutes(new RouteBuilder() {
            public void configure() {
//            	String googleNewsURL = "https://news.google.com/?output=rss";
            	String googleNewsURL = "https://news.google.com/?output=rss&splitEntries=false";
            	List<String> cities = Arrays.asList(
            			"New York", "Los Angeles", "Chicago", "Miami", "Orlando",
            			"Paris", "Berlin", "Madrid", "Lisbon", "Barcelona",  "Rio de Janeiro",
            			"São Paulo", "Belo Horizonte", "Salvador", "Austin", "Las Vegas",
            			"San Francisco", "Ohio", "Michigan", "Vancouver", "Toronto",
            			"Mexico City"
            	);
            	String regexPattern = "(?i).*( " + String.join("|", cities) + " ).*";

            	from("rss:" + googleNewsURL)
//            	from("rss:" + googleNewsURL  + "?delay=1000")
	                .log("Received RSS feed")
	                .marshal().rss()
	                .marshal().string()
//	                .log("Marshalled RSS content to string")
	                .split(xpath("/rss/channel/item/title/text()"))
//	                .log("Splitting items based on titles")
	                .setBody(body())
	                .filter(body().regex(regexPattern))
	                .process(new Processor() {
	                    public void process(Exchange e) throws Exception {
	                        String originalBody = e.getIn().getBody(String.class);
	                        String[] parts = originalBody.split(" - ");
	                        String title = parts.length > 0 ? parts[0] : "Title Not Available";
	                        String source = parts.length > 1 ? parts[1] : "Source Not Available";
	                        e.getOut().setBody("Title: " + title + "; Source: " + source);
	                    }
	                })
//	                .to("log:Received ${body}")
	                .to("log:GOOGLE_NEWS_UPDATES RECEIPT?showExchangeId=false&showExchangePattern=false&showBodyType=true")
//	                .to("log:Received?showBody=true&showHeaders=true")
//	                .to(LoggingLevel.DEBUG, "Received", true, true, false, false, false, false, false, false, false)
//	                .to("log:Received?showBody=true&amp;showHeaders=true")
	                .to("file:data/outbox")
	                .to("jms:queue:RSS_GOOGLE_NEWS_UPDATES");
            	
                try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
            }
        });

        // start the route and let it do its work
        context.start();
        Thread.sleep(10000);

        // stop the CamelContext
        context.stop();
    }
}

```

In this code each part is responsible for executing one part of the instructions of the lab:

Accessing and receiving feeds from Google News:
```java
String googleNewsURL = "https://news.google.com/?output=rss&splitEntries=false";
from("rss:" + googleNewsURL)
```


Setting the message body using XPath
```java
.marshal().rss()
.marshal().string()
.split(xpath("/rss/channel/item/title/text()"))
.setBody(body())
```

Filtering incoming feeds based on regular expressions:
```java
List<String> cities = Arrays.asList(
    "New York", "Los Angeles", "Chicago", "Miami", "Orlando",
    "Paris", "Berlin", "Madrid", "Lisbon", "Barcelona", "Rio de Janeiro",
    "São Paulo", "Belo Horizonte", "Salvador", "Austin", "Las Vegas",
    "San Francisco", "Ohio", "Michigan", "Vancouver", "Toronto",
    "Mexico City"
);
String regexPattern = "(?i).*( " + String.join("|", cities) + " ).*";
.filter(body().regex(regexPattern))
```

Logging and routing output to a file:
```java
.to("log:GOOGLE_NEWS_UPDATES RECEIPT?showExchangeId=false&showExchangePattern=false&showBodyType=true")
.to("file:data/outbox")
```

Processing the message to extract title and source:
```java
.process(new Processor() {
    public void process(Exchange e) throws Exception {
        String originalBody = e.getIn().getBody(String.class);
        String[] parts = originalBody.split(" - ");
        String title = parts.length > 0 ? parts[0] : "Title Not Available";
        String source = parts.length > 1 ? parts[1] : "Source Not Available";
        e.getOut().setBody("Title: " + title + "; Source: " + source);
    }
})
```

Logging and routing output to a file:
```java
.to("log:GOOGLE_NEWS_UPDATES RECEIPT?showExchangeId=false&showExchangePattern=false&showBodyType=true")
.to("file:data/outbox")
```


When we run the file from terminal:

```
mvn exec:java -Dexec.mainClass=camelinaction.FileCopierWithCamel
```

The log output is as follows:

```
11:54:23,115 DEBUG [SendProcessor] >>>> Endpoint[log://GOOGLE_NEWS_UPDATES%20RECEIPT?showBodyType=true&showExchangeId=false&showExchangePattern=true] Exchange[Message: Title: Drake's Security Guard Shot Outside His Toronto Home, Rapper Uninjured; Source: Rolling Stone]
```

The output truncated:

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/fernandorochacorreaurbano/.m2/repository/org/slf4j/slf4j-log4j12/1.7.5/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/fernandorochacorreaurbano/.m2/repository/org/apache/activemq/activemq-all/5.9.0/activemq-all-5.9.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
11:54:20,045 DEBUG [DefaultCamelContext] Adding routes from builder: Routes: []
11:54:21,113 INFO  [DefaultCamelContext] Apache Camel 2.12.3 (CamelContext: camel-1) is starting
11:54:21,114 DEBUG [DefaultCamelContext] Using ClassResolver=org.apache.camel.impl.DefaultClassResolver@211a1cbd, PackageScanClassResolver=org.apache.camel.impl.DefaultPackageScanClassResolver@22afea4a, ApplicationContextClassLoader=org.codehaus.mojo.exec.URLClassLoaderBuilder$ExecJavaClassLoader@776ac631
11:54:21,114 INFO  [ManagedManagementStrategy] JMX is enabled
11:54:21,137 DEBUG [DefaultManagementAgent] Starting JMX agent on server: com.sun.jmx.mbeanserver.JmxMBeanServer@5aa3a8de
11:54:21,155 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=context,name="camel-1"
11:54:21,155 DEBUG [TimerListenerManager] Added TimerListener: org.apache.camel.management.mbean.ManagedCamelContext@5617b2a4
11:54:21,155 DEBUG [DefaultManagementLifecycleStrategy] Registering 1 pre registered services
11:54:21,156 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=components,name="jms"
11:54:21,165 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=DefaultTypeConverter
11:54:21,205 INFO  [DefaultTypeConverter] Loaded 251 type converters
11:54:21,206 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=EndpointRegistry
11:54:21,207 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=DefaultExecutorServiceManager
11:54:21,207 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=SharedProducerServicePool
11:54:21,207 DEBUG [SharedProducerServicePool] Starting service pool: org.apache.camel.impl.SharedProducerServicePool@3b0d5592
11:54:21,207 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=DefaultInflightRepository
11:54:21,208 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=DefaultShutdownStrategy
11:54:21,208 DEBUG [DefaultManagementAgent] Registered MBean with ObjectName: org.apache.camel:context=Fernandos-MacBook-Pro.local/camel-1,type=services,name=DefaultPackageScanClassResolver
11:54:21,212 DEBUG [DefaultCamelContext] Using ComponentResolver: org.apache.camel.impl.DefaultComponentResolver@6d5e67f7 to resolve component with name: rss
11:54:21,212 DEBUG [DefaultComponentResolver] Found component: rss in registry: null
[continues...]
```

![](camel-messages-from-rss/assets/lab7-01.png)

The messages are also sent to data/outbox:

```
(base) fernandorochacorreaurbano@Fernandos-MacBook-Pro Producer % cd data/outbox 
(base) fernandorochacorreaurbano@Fernandos-MacBook-Pro outbox % ls
ID-Fernandos-MacBook-Pro-local-50900-1715037911339-0-16	ID-Fernandos-MacBook-Pro-local-54992-1715088770771-0-22	ID-Fernandos-MacBook-Pro-local-55472-1715090842009-0-36	ID-Fernandos-MacBook-Pro-local-56051-1715094349283-0-11
ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-21	ID-Fernandos-MacBook-Pro-local-54992-1715088770771-0-31	ID-Fernandos-MacBook-Pro-local-55498-1715091005364-0-12	ID-Fernandos-MacBook-Pro-local-56051-1715094349283-0-18
ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-25	ID-Fernandos-MacBook-Pro-local-54992-1715088770771-0-35	ID-Fernandos-MacBook-Pro-local-55498-1715091005364-0-28	ID-Fernandos-MacBook-Pro-local-56051-1715094349283-0-23
ID-Fernandos-MacBook-Pro-local-50943-1715038095004-0-30	ID-Fernandos-MacBook-Pro-local-55000-1715088790156-0-11
[continues...]
```

Inside each message, we can see that the format:

![](camel-messages-from-rss/assets/lab7-02.png)

