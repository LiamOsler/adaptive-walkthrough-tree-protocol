ReadMe.txt
Author: Liam Osler
Date: March 19th 2021

The main method of the program is called "Main".

Compilation instuctions - In the project directory, compile the program using the javac command:


        javac Main.java
        
        
        
Running the program - once compiled the program can be run with the following command:
  
  
        java Main
        
        
        
The program will execute and the output displayed in the console like so:

        The number of stations in the network under investigation is: 8
        The stations that have a frame to send are: C,E,F,H
        Timeslot 0: C, E, F, H => Conflict
        Timeslot 1: C => Success
        Timeslot 2: E, F, H => Conflict
        Timeslot 3: E, F => Conflict
        Timeslot 4: E => Success
        Timeslot 5: F => Success
        Timeslot 6: H => Success

