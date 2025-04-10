Gradle Hello World Starter
A template to start a new Gradle project from.

Exercise
Please follow the instructions to get started with the Gradle Exercise. In this exercise we will build a Java CLI application that prints greets the user in different languages and outputs the text in different colors. We will display use Gradle to build the application, manage resources and manage dependencies.

1. Download the repository
Download this repository's zip file and extract it to a folder on your computer.
Open the folder in IntelliJ IDEA.
Run the org.example.Main class to verify that the application runs. It should print Hello World! to the console.
Initialize a new Git repository in the folder by running git init in the terminal.
Use git add . to add all files to the staging area.
Use git commit -m "Initial commit" to commit the files to the repository.
Create a new repository on GitHub and follow the instructions on Github to link it to your local repository.
Push your local repository to the remote repository on Github.
Verify that your files are on Github.
1.1 Build the application
To build the application we need to run the build task. Run the following command in the terminal to build the application:

./gradlew build
1.1.1 Setup Proxy if working from the University Lab (Optional)
This step is necessary only if you are doing the exercise from the University computers or if the computer you are using is connected to the University network.

Please edit the gradle.properties file and add the required proxy and port that will be provided by the teaching assistant!

1.2 Run the application
We can run the application by running the org.example.Main class:

cd build/classes/java/main
java org.example.Main
This should print out Hello World! to the console.

2. Develop & Build the Hello Gradle application
First, we need to see how we can color the text that we print to the console. We will use the ANSI escape codes to color the text. In Java we can use Jansi library to print colored text to the console.

2.1. Add Jansi as a dependency
To add the Jansi library as a dependency we need to add the following lines to the build.gradle file: To find the Jansi library on Maven Central we can search for jansi. The results will get you to this link, from where you can also copy the dependency declaration:

dependencies {
    //  You could also use the shorter notation:
    //  implementation 'org.fusesource.jansi:jansi:2.4.0'
    implementation group: 'org.fusesource.jansi', name: 'jansi', version: '2.4.0'
}
2.2. Use Jansi to print colored text
To use Jansi we need to import the org.fusesource.jansi.AnsiConsole class and call the systemInstall() method. This will install the Jansi library and allow us to use the Ansi class to print colored text to the console. In the org.example.Main class, add the following lines to the main method:

package org.example;

import org.fusesource.jansi.Ansi;
import org.fusesource.jansi.AnsiConsole;

public class Main {

    public static void main(String[]args){
        AnsiConsole.systemInstall();
        System.out.println(Ansi.ansi().fg(Ansi.Color.RED).a("Hello World!").reset());
        AnsiConsole.systemUninstall();
    }
}
This will print the text Hello World! in red to the console. However, running the code with IntelliJ will not print the text in red. We need to run it from the terminal. To run the application from the terminal, we need to build it first. We will use Gradle to build the application. Follow the instructions in the next section to build the application.

2.3. Build the application
To build the application we need to run the build task. Run the following command in the terminal to build the application:

./gradlew build
This will create a build folder in the project's root folder. The build folder will contain the compiled classes and the JAR file. The JAR file will be located in the build/libs folder. The name of the JAR file will be hello-gradle-0.1.0-SNAPSHOT.jar. The build/classes/java/main folder will contain the compiled classes. The org.example.Main class will be located in the org/example folder.

2.4 Run the application
We can run the application in multiple different ways:

2.4.1 Running the Main class
We can run the application by running the org.example.Main class:

cd build/classes/java/main
java org.example.Main
This will however produce an error message:

Error: ClassNotFoundException org.fusesource.jansi.AnsiConsole not found
This is because the Jansi library is not on the classpath. We can fix this by adding the required JAR files to the classpath, which we will do in the next section

2.4.2 We can run the application by running the JAR file directly:
java -jar build/libs/hello-gradle-0.1.0-SNAPSHOT.jar
However, this will display an error message:

no main manifest attribute, in build/libs/hello-gradle-0.1.0-SNAPSHOT.jar
This is because the JAR file does not contain the Main-Class attribute in the MANIFEST.MF file. We can find the MANIFEST.MF file in the build/tmp/jar folder, or in the META-INF folder in the JAR file. We can fix this by adding the following lines to the build.gradle file:

jar {
    manifest {
        attributes 'Main-Class': 'org.example.Main'
    }
}
Now, when we run the ./gradlew build task, the MANIFEST.MF file will be updated with the Main-Class attribute.

Try running the application again:

java -jar build/libs/hello-gradle-0.1.0-SNAPSHOT.jar
However this will produce the same error message as before, becuase Jansi is not on the classpath:

Error: ClassNotFoundException org.fusesource.jansi.AnsiConsole not found
To add the Jansi library to the classpath, we can add the following lines to the build.gradle file:

jar {
    manifest {
        attributes 'Main-Class': 'org.example.Main'
    }
    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
This will create a fat JAR file, which contains all the dependencies.

Try building and then running the application again:

./gradlew build
java -jar build/libs/hello-gradle-0.1.0-SNAPSHOT.jar
This will print Hello World! to the console (in red).

2.4.3 Running the application with Gradle run task
We can also run the application by using the run task:

./gradlew run
This will however fail, because the run task does not exist in the context of the build.gradle file. We can fix this by adding the following lines to the build.gradle file:

plugins {
    id 'java'
    // Apply the application plugin to add support for building a CLI application in Java.
    id 'application'
}

application {
    // Define the main class for the application.
    mainClassName = 'org.example.Main'
}
By adding the application plugin, we are able to use the run task. We are also configuring the run task to use the org.example.Main class as the main class of the application.

Now, when we run the ./gradlew run task, the application will be run.

This will print Hello World! to the console (but NOT in red).

2.4.4 Cleaning the build.gradle file
We now use the org.example.Main class as the main class of the application in two places already:

in the application plugin configuration
in the jar task configuration
We can refactor the build.gradle file to use a variable to specify the main class of the application. We can add the following lines to the build.gradle file:

def mainClassName = 'org.example.Main'
We can now use the mainClass variable in the application plugin configuration and in the jar task configuration:

application {
    // Define the main class for the application.
    mainClass = mainClassName
}
jar {
    manifest {
        attributes 'Main-Class': mainClassName
    }

    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
Instead of defining a local variable, we could also define the mainClassName variable in the gradle.properties file, located in the project's root folder:

mainClassName=org.example.Main
Now verify that the application still works by running the ./gradlew run task.

./gradlew run
Also Verify that you can build and run your application:

./gradlew build
java -jar build/libs/hello-gradle-0.1.0-SNAPSHOT.jar
2.5. Develop Multi-Language Support in the Application
We are going to use the first parameter of the application to specify the language that the application should greet the user in.

A greeting will be defined by three components: the greeting (e.g. Hello), the person (e.g. World) and the punctuation (e.g. !). Each component will be able to specify a text and a color. The application will print the greeting in the specified color. We will use *.properties files to specify the text and color for each component. These files will be located in the src/main/resources folder. We will name the files ro.properties, en.properties, es.properties and de.properties to specify the language of the greeting.

Create the following files in the src/main/resources folder:

de.properties:
greeting.text=Hallo
greeting.color=black
person.text=Freunde
person.color=yellow
punctuation.text=?!
punctuation.color=red
en.properties:
greeting.text=Hello
greeting.color=red
person.text=friends
person.color=cyan
punctuation.text=!
punctuation.color=default
es.properties:
greeting.text=Ola
greeting.color=red
person.text=amigos
person.color=magenta
punctuation.text=!!!
punctuation.color=red
ro.properties:
greeting.text=Salutare
greeting.color=red
person.text=prieteni
person.color=yellow
punctuation.text=!!!
punctuation.color=blue
Create a PropertiesUtils class in the org.example package. This class will be used to load the properties files and get the values for the components of the greeting.

package org.example;

import java.util.Properties;

public class PropertiesUtils {

    public static Properties getPropertiesForLanguage(String language) {
        Properties properties = new Properties();
        try {
            properties.load(PropertiesUtils.class.getResourceAsStream("/" + language + ".properties"));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return properties;
    }
}
Next create a file called Greeting.java in the org.example package. This class will be used to store the components of the greeting.

package org.example;

import org.fusesource.jansi.Ansi;

import java.util.Properties;

public class Greeting {

    private final Properties properties;

    public Greeting(String language) {
        properties = PropertiesUtils.getPropertiesForLanguage(language);
    }

    public String getGreeting() {
        return properties.getProperty("greeting.text");
    }

    public Ansi.Color getGreetingColor() {
        return getAnsiColor("greeting.color");
    }

    public String getPerson() {
        return properties.getProperty("person.text");
    }

    public Ansi.Color getPersonColor() {
        return getAnsiColor("person.color");
    }

    public String getPunctuation() {
        return properties.getProperty("punctuation.text");
    }

    public Ansi.Color getPunctuationColor() {
        return getAnsiColor("punctuation.color");
    }

    private Ansi.Color getAnsiColor(String PERSON_COLOR_PROPERTY) {
        try {
            return Ansi.Color.valueOf(properties.getProperty(PERSON_COLOR_PROPERTY).toUpperCase());
        } catch (IllegalArgumentException e) {
            return Ansi.Color.DEFAULT;
        }
    }
}
The Greeting class will contain all the fields that a language can define for a greeting. The getAnsiColor method will convert the color name to an Ansi.Color enum value.

Lastly, we need to update the Main class to use the Greeting class.

package org.example;

import org.fusesource.jansi.Ansi;
import org.fusesource.jansi.AnsiConsole;

public class Main {
    public static void main(String[] args) {
        AnsiConsole.systemInstall();

        String language;

        if (args.length == 0) {
            language = "en";
        } else {
            language = args[0].toLowerCase();
        }
        Greeting gp = new Greeting(language);

        System.out.println(Ansi.ansi()
                .fg(gp.getGreetingColor()).a(gp.getGreeting() + " ")
                .fg(gp.getPersonColor()).a(gp.getPerson() + " ")
                .fg(gp.getPunctuationColor()).a(gp.getPunctuation())
                .reset()
        );

        AnsiConsole.systemUninstall();
    }
}
By default, the application will use the en language. If a language is specified as a parameter, the application will use that language.

Verify that the application works by running the ./gradlew run task. Use all languages as parameters and see if the application prompts the correct greeting:

./gradlew run --args="ro"
Also verify that the application works by building and running the jar file:

./gradlew build
java -jar build/libs/hello-gradle-0.1.0-SNAPSHOT.jar ro
3. Install a distribution of the application and be able to run the project from the command line
To install a distribution of the application, we will use the application plugin, which uses the distribution plugin:

./gradlew installDist
This will create a build/install folder, which will contain the distribution of the application. The build/install folder will contain the following structure:

build/install
└── hello-gradle
     ├── bin
     │    ├── hello-gradle
     │    └── hello-gradle.bat
     └── lib
          └── hello-gradle-0.1.0-SNAPSHOT.jar
Add the build/install/hello-gradle/bin folder to the PATH environment variable. To do this temporarily, you can run:

export PATH=$PATH:$(pwd)/build/install/hello-gradle/bin
This will allow us to run the application from the command line by just using it's name.

hello-gradle
