---
layout: gradle
title: Nodeclipse/Enide Gradle for Eclipse - Android
---

# Gradle for Eclipse - Android

<p></p>

Back to [Gradle for Eclipse](../) page

Pages:

- [Drop-in template for classic Android project](https://github.com/Nodeclipse/nodeclipse-1/blob/master/org.nodeclipse.enide.editors.gradle/docs/android/build.gradle)
- [Importing from Android Studio into Eclipse](android/Importing-from-Android-Studio-into-Eclipse)
- see AAR in Eclipse below

### Command line

Android SDK is enough to create and build Android app. No GUI, just CLI:

	android create project -p AppPAKT -a MainActivity -k com.example.apppaktgv -t android-19

or with new (Maven) layout and gradle files:

	android create project -p AppPAKTGV -a MainActivity -k com.example.apppaktgv -t android-19 -g -v 0.12.+

### Download Eclipse ADT

see [developer.android.com - Installing the Eclipse Plugin](https://developer.android.com/sdk/installing/installing-adt.html)

use URL  
`https://dl-ssl.google.com/android/eclipse/`    
or use offline installing from zip archive e.g. [ADT-23.0.4.zip](https://dl.google.com/android/ADT-23.0.4.zip)

### AAR

Taken from <http://stackoverflow.com/questions/16709305/add-dependencies-via-gradle-to-eclipse-in-android-project?rq=1>

My (@JavaJedi) solution is based off Rafael's above in that it copies dependencies to the libs directory which is only used by Android. However I go further to completely explode the referenced AAR's for use in Eclipse.

#### Gradle Build File

Add the following to the end of your Android projects build.gradle :

    task copyJarDependencies(type: Copy) {
        description = 'Used for Eclipse. Copies all dependencies to the libs directory. If there are any AAR files it will extract the classes.jar and rename it the same as the AAR file but with a .jar on the end.'
        libDir = new File(project.projectDir, '/libs')
        println libDir
        println 'Adding dependencies from compile configuration'
        configurations.compile.filter {it.name.endsWith 'jar'}.each { File file -> moveJarIntoLibs(file)}
        println 'Adding dependencies from releaseCompile configuration'
        configurations.releaseCompile.filter {it.name.endsWith 'jar'}.each { File file -> moveJarIntoLibs(file)}
        println 'Adding dependencies from debugCompile configuration'
        configurations.debugCompile.filter {it.name.endsWith 'jar'}.each { File file -> moveJarIntoLibs(file)}
        println 'Adding dependencies from instrumentTestCompile configuration'
        configurations.instrumentTestCompile.filter {it.name.endsWith 'jar'}.each { File file -> moveJarIntoLibs(file)}
        println 'Extracting dependencies from compile configuration'
        configurations.compile.filter {it.name.endsWith 'aar'}.each { File file -> moveAndRenameAar(file) }
        println 'Extracting dependencies from releaseCompile configuration'
     	configurations.releaseCompile.filter {it.name.endsWith 'aar'}.each { File file -> moveAndRenameAar(file) }
     	println 'Extracting dependencies from debugCompile configuration'
     	configurations.debugCompile.filter {it.name.endsWith 'aar'}.each { File file -> moveAndRenameAar(file) }
     	println 'Extracting AAR dependencies from instrumentTestCompile configuration'
     	configurations.instrumentTestCompile.filter {it.name.endsWith 'aar'}.each { File file -> moveAndRenameAar(file) }
    }
    
    void moveJarIntoLibs(File file){
    	println 'Added jar ' + file
            copy{
                from file
                into 'libs'
            }
    }
    
    void moveAndRenameAar(File file){
    	println 'Added aar ' + file
        def baseFilename = file.name.lastIndexOf('.').with {it != -1 ? file.name[0..<it] : file.name}
        
        // directory excluding the classes.jar
        copy{
        	from zipTree(file)
        	exclude 'classes.jar'
            into 'libs/'+baseFilename
        }
        
        // Copies the classes.jar into the libs directory of the expoded AAR.
        // In Eclipse you can then import this exploded ar as an Android project
        // and then reference not only the classes but also the android resources :D 
        copy{
            from zipTree(file)
            include 'classes.jar'
            into 'libs/' + baseFilename +'/libs'
            rename { String fileName ->
                fileName.replace('classes.jar', baseFilename + '.jar')
            }
        }
    }

#### Building with Gradle

Run `gradle clean build` ( or within Eclipse right-click `build.gradle` Rus As -> Gradle build )

You should find all dependencies and exploded AARs in your libs directory. This is all Eclipse should need.

#### Importing in Eclipse

Now this is where the real benefit begins. After you've generated the libs directory from the gradle step above you'll notice there are folders in there too.
 Those new folders are the exploded AAR dependencies from your `build.gradle` file. 

Now the cool part is that when you import your existing Android project into Eclipse it will also detect the exploded AAR folders as projects it can import too! 

**1.** Import those folders under your project's **libs** directory, *don't import any 'build' folders, they're generated by Gradle*

**2.** Ensure you perform a ***Project -> Clean*** on all AAR projects you've added. In your workspace check that each AAR exploded project has the following in the project.properties :

    target=android-<YOUR INSTALLED SKD VERSION GOES HERE>
    android.library=true

**3.** Now in your main Android project you can just add the library references with either ADT or you can just edit the project.properties file and add 

`android.libraries.reference.1=libs/someExplodedAAR/`

**4.** Now you can right-click on your main Android project and **Run as -> Android Application**.


#### But what does this even mean?

1. Well it means you don't need the source code for any of your Android AAR Gradle dependencies in order to reference both it's classes and resources in Eclipse. 

2. The gradle build script above takes the AAR file and prepares it for use in Eclipse. Once you add it to your workspace you're ready to just focus on your actual main Android project. 

3. You can now debug and develop using Eclipse and deploy using ADT with AAR dependencies being properly bundled in the APK. When you need to make some specific builds then you can use gradle.

[Edit this page](https://github.com/Nodeclipse/www.nodeclipse.org/blob/gh-pages/projects/gradle/android.md)