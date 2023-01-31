# **CSE 15L LAB REPORT 2**
> See [Report Guidelines](https://ucsd-cse15l-w23.github.io/week/week3/#week3-lab-report) for further details

 This report is developed by Zichen Wang.

## Part One: A String Webserver
Based on the lab and lectures from week 2 and related course materials and codes, 
the StringServer.java takes the string as input and provide continuous update of contents.

> The server.java will not be included since it's the application from the course content in week 2 and takes too much space.

```
#StringServer.java
/*  
    Notes: 
    This part of code is developed under the course material codes from professor Politz in week2.
    See https://github.com/zzzwww000/cs15l-wavelet for further details
    The Server.java, according to Edstem post https://edstem.org/us/courses/31260/discussion/2465953
    is directly used from Server.java in the material
*/
import java.io.IOException;
import java.net.URI;
class StringHandler implements URLHandler {
    // This handler adds the strings provided to our websites based on requests
    // Modified based on course materials
    String output = "";
    public String handleRequest(URI url) {
        //no actual inputs
        if (url.getPath().equals("/")) {
            return String.format(output);
        } else {
            //to add more strings
            System.out.println("Path: " + url.getPath());
            if (url.getPath().contains("/add-message")) {
                String[] fetchedFromInput = url.getQuery().split("=");
                if (fetchedFromInput[0].equals("s")) {
                    //if there exists something in input
                    if(fetchedFromInput.length != 1){
                        output += fetchedFromInput[1] + "\n";
                        return String.format(output);
                    }
                    //if input is empty
                    else{
                        return String.format("You have provided nothing in your input, please check.");
                    }
                }
            }
            return "404 Not Found!";
        }
    }
}
class StringServer {
    public static void main(String[] args) throws IOException {
        if(args.length == 0){
            System.out.println("Missing port number! Try any number between 1024 to 49151");
            return;
        }
        int port = Integer.parseInt(args[0]);
        Server.start(port, new StringHandler());
    }
}
```
After saving the codes, use following commands to compile and start the server.

`javac StringServer.java Server.java` and `java StringServer [number]`

In web explorer (in this report, it's Chrome), there are two screenshots that show the result of requests.

**ScreeenShot 1**

![1-1](https://user-images.githubusercontent.com/120359926/215620407-1c4cf60e-a2ee-4183-8f34-1e57b63fbaaa.png)

Here, after calling the main method, the first method is starting server, which takes two parameters: integer for server port and a url handler type (here is string)

In StringServer, it calls the handleRequest method, and takes the url (what we typed in browser) as the input. Also it establishes a value named "output" to store our changes of strings we want to show in our webpage.

In handleRequest, it take the path and check the path contents. When there is something other than a single "/", it checks whether the path contains the request to "add-address".
If there is such request, it uses .split() to separate the parts of the query and fetch the second part (the string that will be inserted) as "fetched From Input", which is also the value we want to insert.

Then, it adds the "fetchedFromInput" and a "/n" (as the tool to split lines) to the output, thus the output is added with our recent changes of strings.

Eventually it will return the output to our webpage.

**ScreeenShot 2**

![1-2](https://user-images.githubusercontent.com/120359926/215620430-2102d6c0-fca9-4e77-81f4-555c520041bd.png)

Here, we have changed the url value. Thus, we have also changed the value of "fetched From Input", the string value that we want to add. By changing the string to add, we also add the string and "/n" to our "output" value, which is what we will see in our webpage.

What does not change is the port number (since we will always work on this port).


## Part Two: Bugs

There are many bugs from week 3 lab codes, for example this one:

```
  static double averageWithoutLowest(double[] arr) {
    if(arr.length < 2) { return 0.0; }
    double lowest = arr[0];
    for(double num: arr) {
      if(num < lowest) { lowest = num; }
    }
    double sum = 0;
    for(double num: arr) {
      if(num != lowest) { sum += num; }
    }
    return sum / (arr.length - 1);
  }
```
According to description, "the code averages the numbers in the array (takes the mean), but leaves out the lowest number when calculating. Returns 0 if there are no elements or just 1 element in the array."

**A basic test**

---
First, we run a simple test to see whether it will work.

```
    @Test
  public void testAverage() {
    double[] input1 = {1.0,4.0,3.0,2.0};
    //double[] input1 = {1.0,4.0,3.0,2.0,1.0,1.0,1.0};
    double ans = 3.0;
    assertEquals(ans, ArrayExamples.averageWithoutLowest(input1),0.001);
  }
```
Here is the output:

![2-0](https://user-images.githubusercontent.com/120359926/215626339-db6447ca-67e0-465c-82e1-3dc50ad6f8e3.png)

It seems that "it just works".

**A test for bugs**

---
Now, we run another test that tests whether it will run as our expectation:

```
   @Test
  public void testAverage() {
    double[] input1 = {1.0,4.0,3.0,2.0,1.0,1.0,1.0};
    double ans = 3.0;
    assertEquals(ans, ArrayExamples.averageWithoutLowest(input1),0.001);
  }
```
In This test, we expect the method remove all the 1.0s in the code and only calculate the average of remaining, whose result should be 3.0.

However, here is the result:

![2-1](https://user-images.githubusercontent.com/120359926/215622453-edded65e-eccc-40b7-aa7f-c450121446a0.png)

As we can see, the actual result here is 1.5, which is not expected.

**Analysis and debugging**

---
In fact, the method itself is problematic at how it tracks what we will take into consideration when summing up. 
`if(num != lowest) { sum += num; }` means while iterating through the array containing all the numbers, it will add all the non-lowest numbers together. 
However, ` return sum / (arr.length - 1);` actually returns the sum divided by **the size of array - 1**, which means it will take the amount of all lowest numbers permanantly as 1. And this is the buggy place. 

In the test we provided above, it only removes a single 1 in counting process, but the sum is `4.0+3.0+2.0 = 9.0`. So the result is `9.0/6 = 1.5`.

To deal with this bug, simply keeping a record of the numbers that are taken into the sum will be fine.

```
    int counted = 0;//records the numbers
    for(double num: arr) {
      if(num != lowest) { sum += num; counted += 1;}
    }
    return sum / (counted);
  }
```

Here, I added a value named `count`, which counts how many numbers will be used to calculate the average.
In the loop, it keeps itself adding one as long as the number is not the lowest. (while sum keeps adding number to itself)
And eventually `return sum / (counted);` will correctly calculate the average by dividing the correct count of numbers and return.

**Testing the solution**

---
Here is the test result:

![2-2](https://user-images.githubusercontent.com/120359926/215623944-929172d2-0b63-4a4e-8007-e9908af4e9d8.png)

As we can see, the code has passed the test and functions as expected.
Here is a comparison between two codes: (left: original, right: modified)

![2-4](https://user-images.githubusercontent.com/120359926/215624693-1d20fdff-54a7-4f46-9fa7-aa70d9883d8c.png)
![2-3](https://user-images.githubusercontent.com/120359926/215624690-c6364964-2e1a-4a20-9a0a-b2511a448fbb.png)

## Part Three: Reflections
One of the key ideas I have learnt is the process of determining bugs, symptoms and the process of debugging.
To determine why the program is not functioning as expected, writing some helper methods for testing will significantly assist this process. Then, read through these tests and think carefully about why certain symptoms may mark an error. 
Also, developing good coding habits like always leave some in-line comments in important parts of code and keep tracking the values in the process will be helpful in future.

