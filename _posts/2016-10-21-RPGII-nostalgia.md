---
layout: post
title: RPG II nostalgia with Java
---
Way back at the beginning of my career, the first system I was introduced to was an [IBM System/3][1] and the programming language of choice was [RPG II][2]. RPG is short for for Report Program Generator, and that was it was all about initially - reading in a stack of punched cards as input and producing some sort of printed report. By the time I came to use it, input usually came from magnetic media of some kind, but the program flow was driven by the same built-in cycle. In latter years, the language evolved somewhat beyond it's original purpose. In latter years, the language evolved somewhat beyond it's original purpose.

To cut a long story short, a couple of days ago, I was googling for something-or-other, and quite by chance, I came across a simplified flowchart for the RPG Program Cycle, which is essentially just a read loop over a file with a few distinct processes.

### The RPG Program Cycle

![RPG Program Cycle][3]

I was almost overcome with nostalgia (not really) and I decided there and then that it would be fun to replicate such cycle in Java. Now before anyone gets too excited, I didn't want to invest a whole lot of time in this endeavour, so I only made a very basic implementation that emulated the processing of a single input file (Primary File in RPG terms) and producing a report.

Just in case the link to the flowchart above gets broken, here are the main steps in the RPG cycle:-

Start

1. Write heading and detail lines
2. Get input record
3. Perform total calculations
4. Write total output
5. Last record? yes=exit, no=continue
6. Move fields
7. Perform detail calculations

Repeat ad-infinitum

Although it wasn't strictly necessary I decided to write a Java interface `RpgCycle` for the 7 steps above, and here is how it looked.
### The RPG Program Cycle Java interface

{% highlight java %}
package rpg;

public interface RpgCycle {
    default void detailOutput() {}
    default Object getInput() {return null;}
    default void totalCalcs() {}
    default void totalOutput() {}
    boolean lastRecord();
    default void moveFields(Object input) {}
    void detailCalcs();
}
{% endhighlight %}

The methods defined reflect the steps numbered peviously and are listed in the same order. Notice that I provided empty default implementations for all but two of the methods. This is because often many of them were not required in the program. What might, on the face, look questionable is the default method for `getInput` The explanation is that later versions of RPG did not require a primary file, and everything would be done in detail calcs..
### The RPG Program abstract class
Next I created an abstract class `RpgProgram` which implemented the above interface, and in that class, I created a `start()` method which looks like this.

{% highlight java %}
protected final void start() throws IOException {
    try (BufferedReader rdr = new BufferedReader(new FileReader(inputPrimary.toFile()))) {
        // now start the RPG cycle...
        for (;;) {
            // (1) write heading and detail lines
            detailOutput();
            firstPage = false;
            // (2) get input record
            final String input = rdr.readLine();
            lastRecord = lastRecord ? true : input == null;
            // (3) perform total calculations
            totalCalcs();
            // (4) write total output
            totalOutput();
            // (5) LR on?
            if (lastRecord()) {
                break;
            }
            // (6) move fields
            moveFields(input);
            // (7) perform detail calculations
            detailCalcs();
        }
    }
}
{% endhighlight %}

There are a few supporting methods and variables which can be seen in the complete listing at the end of this posting, but essentially you can see how all the steps are performed in a continuous loop until the input file has been completely read.
### The Main Program
The main program uses an input CSV file and produces and output (to stdout in this case), and here it is in all it's glory...

{% highlight java %}
package rpg;

import java.io.PrintStream;

public class MyRpgProgram extends RpgProgram {
    private static final String HDR1 = "Line  Name                  Amt (£)";
    private static final String HDR2 = "====  ====================  =======";
    private static final String ROW_FMT = "%4d  %-20s  %,7.2f%n";

    // where the output will be written
    private final PrintStream out = System.out;

    // define the input record
    private int lineNbr;
    private String fname;
    private String lname;
    private double amount;

    private double totalExpenses;

    @Override
    public void detailOutput() {
        if (firstPage()) {
            out.println(HDR1);
            out.println(HDR2);
        } else {
            out.printf(ROW_FMT, lineNbr, fname + " " + lname, amount);
        }
    }

    @Override
    public void totalOutput() {
        if (lastRecord()) {
            out.printf("%nTOTAL AMOUNT: £%,.2f%n", totalExpenses);
        }
    }

    @Override
    public void moveFields(Object input) {
        // split the input string into an array
        final String[] arr = ((String) input).split(",");
        // convert fields to type
        lineNbr = Integer.parseInt(arr[0]);
        fname = arr[1];
        lname = arr[2];
        amount = Double.parseDouble(arr[3]);
    }

    @Override
    public void detailCalcs() {
        totalExpenses += amount;
    }

    public static void main(String[] args) throws Exception {
        new MyRpgProgram().withInputPrimary("res/expenses.csv").start();
    }

}
{% endhighlight %}

And the output looks like this...
{% highlight text %}
Line  Name                  Amt (£)
====  ====================  =======
   1  Lisa Fuller            300.75
   2  Henry Morales          385.94
   3  Lori Gibson            279.16
   4  Kevin Gardner          167.14
   5  Billy Ortiz            215.23
   6  Doris Gardner           55.88
   7  Diana Hunter             6.89
   8  Bonnie Lane            427.79
   9  Rachel Romero          319.36
  10  Henry Crawford         469.02

TOTAL AMOUNT: £2,627.16
{% endhighlight %}
All in all it was quite fun. Of course it's not perfect by any means, but it could easily be made more sophisticated to support more complex processing, and it's actually quite cool for making quick reports, just like the original inspiration was.

#### expenses.csv
{% highlight csv %}
1,Lisa,Fuller,300.75
2,Henry,Morales,385.94
3,Lori,Gibson,279.16
4,Kevin,Gardner,167.14
5,Billy,Ortiz,215.23
6,Doris,Gardner,55.88
7,Diana,Hunter,6.89
8,Bonnie,Lane,427.79
9,Rachel,Romero,319.36
10,Henry,Crawford,469.02
{% endhighlight %}

#### RpgProgram.java
{% highlight java %}
package rpg;

import java.io.*;
import java.net.*;
import java.nio.file.*;
import java.text.MessageFormat;
import java.util.Optional;

public abstract class RpgProgram implements RpgCycle {

    private Path inputPrimary;

    private boolean lastRecord = false;
    private boolean firstPage = true;

    @Override
    public final Object getInput() {
        // we don't use this, but want to prevent sub-class implementation
        return null;
    }

    @Override
    public final boolean lastRecord() { return lastRecord; }

    protected final RpgProgram withInputPrimary(String file) throws FileNotFoundException, URISyntaxException {
        this.inputPrimary = getPath(file);
        return this;
    }

    protected final boolean firstPage() { return firstPage; }

    protected final void setLR() { this.lastRecord = true; }

    protected final void start() throws IOException {
        // check the input primary file
        if (inputPrimary == null) {
            throw new RuntimeException("no input primary file specified");
        }

         try (BufferedReader rdr = new BufferedReader(new FileReader(inputPrimary.toFile()))) {
            // now start the RPG cycle...
            for (;;) {
                // (1) write heading and detail lines
                detailOutput();
                firstPage = false;
                // (2) get input record
                final String input = rdr.readLine();
                lastRecord = lastRecord ? true : input == null;
                // (3) perform total calculations
                totalCalcs();
                // (4) write total output
                totalOutput();
                // (5) LR on?
                if (lastRecord()) {
                    break;
                }
                // (6) move fields
                moveFields(input);
                // (7) perform detail calculations
                detailCalcs();
            }
        }

    }

    private Path getPath(String file) throws FileNotFoundException, URISyntaxException {
        URL url = Optional.ofNullable(getClass().getClassLoader().getResource(file))
                .orElseThrow(() -> new FileNotFoundException(MessageFormat.format("file \"{0}\" not found", file)));
        return Paths.get(url.toURI());
    }

}
{% endhighlight %}

[1]: https://en.wikipedia.org/wiki/IBM_System/3 "IBM System/3"

[2]: https://en.wikipedia.org/wiki/IBM_RPG_II "RPG II"

[3]: https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_61/rzasd/rpglcyc.gif  "RPG Program Cycle"
