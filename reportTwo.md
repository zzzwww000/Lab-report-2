# **CSE 15L LAB REPORT 2**
> See [Report Guidelines](https://ucsd-cse15l-w23.github.io/week/week3/#week3-lab-report) for further details

> This report is developed by Zichen Wang.

## Part One: A String Webserver
Based on the lab and lectures from week 2 and related course materials and codes, the StringServer.java takes the string as input
and provide continuous update of contents

```

# StringServer.java
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
