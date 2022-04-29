# AAPT and aapt2 -- fixed resource ID and public tag
## Preface
The whole article is about tinker Of TinkerResourceIdTask The knowledge points in the .

aapt and aapt2 The difference of （ Running environment and running results ）;
resources id Fixation ;
Conduct PUBLIC The tag ;
aapt The operating environment is gradle:2.2.0 and gradle-wrapper:3.4.1

aapt2 The operating environment is gradle:3.3.2 and gradle-wrapper:5.6.2

android-aapt-sample The project is an example of my own experiment . Yes aapt and aapt2 Two branches , Corresponding to its implementation respectively .

## AAPT summary
from Android Studio 3.0 Start ,google The default is on aapt2 As a compiler for resource compilation ,aapt2 Appearance , Provides support for incremental compilation of resources . Of course, there will be some problems in the process of using , We can pass in gradle.properties Middle configuration android.enableAapt2=false To close aapt2.

## resources
Android Born to be compatible with a wide range of different devices to do a lot of work , Like screen size 、 internationalization 、 keyboard 、 Pixel density and so on , We can do compatibility for a variety of specific scenarios using specific resources without changing a line of code , Suppose we adapt different resources for different scenarios , How can we apply these resources quickly ？Android For us R This class , Specifies the index of a resource （id）, Then we just need to tell the system that in different business scenarios , Just use the corresponding resources , As for the specific file in the specified resource , It is decided by the system according to the configuration of the developer .

In this case , Suppose we give id yes x value , Now when the business needs to use this resource , The state of the phone is y value , With (x,y), In a table, you can quickly locate the specific path of the resource file . This watch is resources.arsc, It's from aapt compiled .

In fact, binary resources （ Such as the picture ） There is no need to compile , It's just this “ compile ” act , It's to generate resources.arsc And right xml File binary operation ,resources.arsc It's the watch above ,xml The binarization of is for better performance in system reading .AssetManager After we call R dependent id When , You will find the corresponding file in this table , Read out .

Gradle In the process of compiling resources , That's what's called aapt2 command , The parameters are also described in this document , It's just hiding the call details from the developers .

aapt2 There are mainly two steps , One step is compile, One step is link.

Create an empty project ： Only two xml, Namely AndroidManifest.xml and activity_main.xml.

### Compile
 mkdir compiled
 aapt2 compile src/main/res/layout/activity_main.xml -o compiled/
 Copy code 
stay compiled In the folder , Generated layout_activity_main.xml.flat This file , It is aapt2 Peculiar ,aapt No, (aapt The copy is the source file ),aapt2 It can be used for incremental compilation . If we have a lot of documents , You need to call... In turn compile Talent , In fact, it can also be used here –dir Parameters , Only this parameter has no effect of incremental compilation . in other words , When passing the entire directory , Even if only one resource has changed ,AAPT2 It will also recompile all the files in the directory .

### Link
link Workload ratio of compile A little more , The input here is multiple flat The file of and AndroidManifest.xml, External resources , The output is resource only apk and R.java. The order is as follows ：
```
aapt2 link -o out.apk \
-I $ANDROID_HOME/platforms/android-28/android.jar \
compiled/layout_activity_main.xml.flat \
--java src/main/java \
--manifest src/main/AndroidManifest.xml
 Copy code 
 ```
The second line -I yes import External resources , Here is mainly android Some properties defined under the namespace , We usually use @android:xxx It's all in this jar Inside , In fact, we can also provide our own resources for others to link to ;
The third line is the input flat file , If there are more than one , It's just right in the back ;
The fourth line is R.java Generated directory ;
The fifth line specifies AndroidManifest.xml;
Link When finished, it will generate out.apk and R.java,out.apk Contains one resources.arsc file . Only with resource file can use suffix .ap_.

Look at the compiled resources
In addition to using Android Studio To view the resources.arsc, It can also be used directly aapt2 dump apk Information to view resource related ID And status ：
```
aapt2 dump out.apk
 Copy code 
 ```
The output is as follows ：

### Binary APK
```
Package name=com.geminiwen.hello id=7f
  type layout id=01 entryCount=1
    resource 0x7f010000 layout/activity_main
      () (file) res/layout/activity_main.xml type=XML
 Copy code 
 ```
You can see layout/activity_main Corresponding ID yes 0x7f010000.

### Resource sharing
android.jar It's just a pile for compiling , When it's actually implemented ,Android OS Provides a runtime library (framework.jar).android.jar It's very much like a apk, It's just that it exists class file , Then there is a AndroidManifest.xml and resources.arsc. That means we can use it as well aapt2 dump, Execute the following command ：
```
aapt2 dump $ANDROID_HOME/platforms/android-28/android.jar > test.out
 Copy code 
 ```
You get a lot of output like this ：
```
resource 0x010a0000 anim/fade_in PUBLIC
      () (file) res/anim/fade_in.xml type=XML
    resource 0x010a0001 anim/fade_out PUBLIC
      () (file) res/anim/fade_out.xml type=XML
    resource 0x010a0002 anim/slide_in_left PUBLIC
      () (file) res/anim/slide_in_left.xml type=XML
    resource 0x010a0003 anim/slide_out_right PUBLIC
      () (file) res/anim/slide_out_right.xml type=XML
 Copy code 
 ```
It's a little more PUBLIC Field of , One apk The resources in the file , If it's marked with this , Can be used by other apk The referenced , The way to quote is @ Package name : type / name , for example ：@android:color/red.

If we want to provide our resources , Well, first of all, lay a foundation for our resources PUBLIC The tag , And then in xml Reference your package name in , such as ：@com.gemini.app:color/red You can refer to what you define color/red 了 , If you don't specify the package name , The default is yourself .


as for AAPT2 How to generate PUBLIC, If you are interested, please read this article .

### ids.xml summary
ids.xml： Provide unique resources for application related resources id.id It's about getting xml Parameters required for objects in , That is to say Object = findViewById(R.id.id_name); Medium id_name.

These values can be used in code android.R.id Quote to . If in ids.xml It defines ID, It's in layout Can be defined as follows @id/price_edit, otherwise @+id/price_edit.

### advantage

It's easy to name , We can name some specific controls first , Direct reference when using id that will do , One naming step is omitted .
Optimize compilation efficiency :
add to id I'll be in R.java In the middle of ;
Use ids.xml Unified management , Compile once and use many times . But use "@+id/btn_next" In the form of , Every time a file is saved (Ctrl+s) after R.java Will be re tested , If the id Then it does not generate , If it doesn't exist, you need to add the id. Therefore, the compilation efficiency is reduced .
ids.xml The contents of the document ：
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="forecast_list" type="id"/>
<!--    <item name="app_name" type="string" />-->
</resources>
 Copy code 
 ```
Some people may be curious that there is a line of annotated code on it , Open the comment and you will find that the compiler will report an error ：
```
Execution failed for task ':app:mergeDebugResources'.
> [string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/strings.xml	[string/app_name] /Users/tanzx/AndroidStudioProjects/AaptDemo/app/src/main/res/values/ids.xml: Error: Duplicate resources
 Copy code 
 ```
because app_name The resources for are already value It's stated in .


## public.xml summary
### Official instructions Official website ： Select the resources you want to make public .
```
Original translation ： All resources in the library are public by default . To make all resources implicitly private , You must define at least one specific attribute as public . Resources include... Of your project res/ All files in directory , Such as images . To prevent users of the library from accessing resources for internal use only , You should use this automatic private identification mechanism by declaring one or more public resources . perhaps , You can also add an empty <public /> Tag makes all resources private , This tag does not make any resources public , It's going to take everything （ All resources ） All private .

By implicitly making properties private , You can not only prevent library users from getting code completion advice from internal library resources , You can also rename or remove private resources , Without destroying the client side of the library . The system filters out private resources from code completion , also Lint A warning is issued when you try to reference private resources .

While building the library ,Android Gradle The plug-in gets the public resource definition , And extract it to public.txt In file , The system then packages the file into AAR In file .
```
