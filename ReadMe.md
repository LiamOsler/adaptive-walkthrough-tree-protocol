# ReadMe.md
# Author: Liam Osler
# Date: March 19th 2021

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

Source
```java
/*
// run.java
// CSCI3171 Assignment 3
// Author: Liam Osler
// Date: March 19th 2021
//
// This program reads the contents of the Input.txt file and a performs an
// Adaptive Tree Walk Protocol, giving the output to the user via console.
*/

//Import required libraries:
import java.nio.file.Files;
import java.nio.file.Path;
import java.io.IOException;
import java.util.ArrayList;

//Node class for the binary tree:
//Each node has a left and right node and parent
//Data is generic and is either an Integer or Char in this contex
class Node<T extends Comparable<T>> {
    Node<T> left;
    Node<T> right;
    Node<T> parent;
    T data;

    public Node(T data) {
        this.data = data;
    }
}

public class Main {
    //Static int for keeping tracking of timeslot from recursive timeslot method:
    static int timeSlot = 0;

    //Alphabet as character array for writing station names:
    static char[] alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();

    //Static ArrayLists used by tree populating method:
    static ArrayList<Node> nodeList = new ArrayList<Node>();
    static ArrayList<Character> stationLetters = new ArrayList<Character>();
    static ArrayList<Integer> stationNumbers = new ArrayList<Integer>();

    //buildTree function:
    //Helper function for recursively building a binary tree, returns the root node
    //Populates a complete binary tree, using the depth of the tree as input
    private static void buildTree(int N) {
        //Populate the stationNumber list with available values
        for(int i = 0; i <= N*2; i++){
            stationNumbers.add(i);
        }
        //Populate the stationLetter list with available values
        for(int i = 0; i <= N; i++){
           stationLetters.add(alphabet[i]);
        }
        //Create the root node:
        Node<Integer> root = new Node<Integer>(stationNumbers.get(0));
        stationNumbers.remove(0);
        addNodes(root, calculateBinaryTreeDepth(N+1));

    }

    //addNodes function:
    //Method for recursively adding nodes to the tree- accepts a parent node and depth
    //Once max depth is reached (leaf level of tree) no more children are added.
    //Uncheck warning surpression required due to use of Raw type parameters (binary values as both char and int)
    @SuppressWarnings("unchecked")
    private static void addNodes(Node curr, int depth) {
        //We have gone down one level of the tree, deprecate depth:
        depth--;
        //Add the current node to the nodeList:
        nodeList.add(curr);
        //If we are above 0 depth, we name the stations only by number:
        if(depth>0) {
            //Create new nodes for the left and right branch and assign their data:
            Node<Integer> left = new Node<Integer>(stationNumbers.get(0));
            //Drop the lead number from the arrayList, so we can increment:
            stationNumbers.remove(0);
            Node<Integer> right = new Node<Integer>(stationNumbers.get(0));
            stationNumbers.remove(0);

            //Call the function recursively:
            addNodes(left, depth);
            addNodes(right, depth);

            //Add the left and right node as the curr node's branch nodes:
            curr.left = left;
            curr.right = right;

            //Add curr node as the parent to the branch nodes:
            left.parent = curr;
            right.parent = curr;
        }

        //If we are at depth 0, we assign a letter to each station:
        else if(depth == 0){
            Node<Character> left = new Node<Character>(stationLetters.get(0));
            //Drop the lead letter from the arrayList, so we can increment to the next:
            stationLetters.remove(0);
            Node<Character> right = new Node<Character>(stationLetters.get(0));
            stationLetters.remove(0);

            //Call the function recursively:
            addNodes(left, depth);
            addNodes(right, depth);

            //Add the left and right node as the curr node's branch nodes:
            curr.left = left;
            curr.right = right;

            //Add curr node as the parent to the branch nodes:
            left.parent = curr;
            right.parent = curr;
        }
    }

    //calculateBinaryTreeDepth function:
    //Calculate the depth of the binary tree, using the formula where N = 2^d, in a complete binary tree
    private static int calculateBinaryTreeDepth(int N){
        return (int) (Math.log(N)/Math.log(2));
    }

    //traceRoute function:
    //Trace the route from the a leaf to the root, return as string array
    private static String[] traceRoute (Node curr){
        //Add the first node to the list:
        StringBuilder route = new StringBuilder("" + curr.data);
        //Move up to the parent node:
        curr = curr.parent;

        //The root of the tree's parent node will be null:
        while(curr.parent != null){
            //For all but the final and first station, we will append a comma:
            route.append(",").append(curr.data);
            curr = curr.parent;
        }

        //Append ,0 to the end of the route, since all routes end at node 0
        route.append(",0");

        //Split the completed string by commas and return String array:
        return route.toString().split(",");
    }

    //findRoutes function, returns an arraylists of string arrays with the paths from the station list
    //to the root node, works in conjuction with the traceRoute function to find the routes of all the stations
    private static ArrayList<String[]> findRoutes(String[] splitStations){
        ArrayList<String[]> stationRoutes = new ArrayList<String[]>();
        //Iterate through all of the array items in the Arraylist:
        for (String splitStation : splitStations) {
            for (Node curr : nodeList) {
                if (splitStation.equals(curr.data.toString())) {
                    //If there is a match to a frame station from the input, we trace it's route to the root:
                    stationRoutes.add(traceRoute(curr));
                }
            }
        }
        return stationRoutes;
    }

    //Get the names as a CSV string (for the Timeslot: ) output
    private static void namesCSV(ArrayList<String[]> routes){
        StringBuilder builder = new StringBuilder();
        for(int i = 0; i<routes.size()-1; i++){
            //Appends only the first item of each route in the route list, giving us the list of stations
            builder.append(routes.get(i)[0]).append(", ");
        }
        builder.append(routes.get(routes.size() - 1)[0]).append(" ");
        System.out.print(builder);
    }


    //frameCheck() function, takes in a stationRoutes list, and the current recursive depth
    //Prints the primary output of interest for the program
    private static void frameCheck(ArrayList<String[]> stationRoutes, int depth){
        //leftRoute will be seperated off from the main stationRoutes, as we add to leftRoute
        ArrayList<String[]> leftRoute = new ArrayList<String[]>();

        //Print out the current time slot and the list of stations in
        System.out.print("Timeslot " + timeSlot + ": ");
        namesCSV(stationRoutes);

        //If there is only one route in the station routes list, we know we have a successful transmission:
        if(stationRoutes.size() == 1){
            //Echo out the success tag:
            System.out.println("=> Success");
            //Increment the timeslot:
            timeSlot++;
        }

        //Otherwise we must split the list:
        else{
            //Echo out the conflict tag:
            System.out.println("=> Conflict");

            //Add the first stations's route to root to the the leftRoute list, and remove it from the remaining list
            leftRoute.add(stationRoutes.get(0));
            stationRoutes.remove(0);

            //We need to check the station to the right of the first left node shares a parent node at this depth:
            for (int i = 0; i < stationRoutes.size()-1; i++) {
                //If the route to the left shares a parent node at this depth with the right
                if(leftRoute.get(0)[depth-1].equals(stationRoutes.get(0)[depth-1])){
                    //Keep adding stationlists to the left side until we reach a node where the parent
                    leftRoute.add(stationRoutes.get(0));
                    stationRoutes.remove(0);
                }
            }
            //Increment the timeslot:
            timeSlot++;
            //Recursively call the function, with the left side being called first
            //Depth -1 because we are moving our check one step down the tree for each call
            frameCheck(leftRoute, depth -1);
            frameCheck(stationRoutes, depth -1);
        }
    }


    public static void main(String[] args) throws IOException{
        //Read the contents of the Input.txt file
        Path inputPath = Path.of("Input.txt");
        String input = Files.readString(inputPath);
        //Split to lines:
        String[] lines = input.split("\\r?\\n");
        //Convert first line to int N, the number of stations- 2, 4, 8, or 16
        //Calculate the depth of the tree
        int N = Integer.parseInt(lines[0]);
        int depth = calculateBinaryTreeDepth(N);
        //Convert the second line to the station list and split to array:
        String stationList = (lines[1]);
        String[] splitStations= stationList.split(",");

        //Output the contents of the Input.txt file to the console:
        System.out.println("The number of stations in the network under investigation is: " + N);
        System.out.println("The stations that have a frame to send are: " + stationList);

        //Build the Binary Tree:
        buildTree(N);

        //Trace the route of each station and write it to an ArrayList:
        ArrayList<String[]> stationRoutes = findRoutes(splitStations);

        //Run the frameCheck() function with station routes as input
        frameCheck(stationRoutes, depth);

    }
}
```
