# Java CookBook Memo

## Chapter 1

* JDK  
	Use the commands javac to compile and java to run your program(and, on Windows only, javaw to run a program without a console window).

* 

## Chapter 3
* StringTokenizer  
	normally breaks the String into tokens at what we would think of as “word boundaries” in European languages.  
  
  Here is the sample code:

	```
	StringTokenizer st = new StringTokenizer("Hello, World|of|Java", ", |",true);	while (st.hasMoreElements( ))		System.out.println("Token: " + st.nextElement( ));
	```
* StringBuilder  
  Strings are immutable. StringBuilders are mutable and designed for, well, building Strings.  

* Stack  
  Stack implements an easy-to-use last-in, first-out(LIFO) stack of objects.  
  
  Here is the sample code:
  	
  	```
	String s = "Father Charles Goes Down And Ends 	Battle";
		// Put it in the stack frontwards	Stack<String> myStack = new Stack<>();	StringTokenizer st = new StringTokenizer(s);
		while (st.hasMoreTokens()) {		myStack.push(st.nextToken());	}
		// Print the stack backwards	System.out.print('"' + s + '"' + " backwards by word 	is:\n\t\"");	while (!myStack.empty()) {		System.out.print(myStack.pop());		System.out.print(' ');	}	System.out.println('"');
	```
  
	