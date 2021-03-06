---
layout: post
title: "IL superset of all .net languages"
date: 2014-01-22 23:14
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [C#, IL]
---


In .net world you can choose your language and write your application with best language which matches your requirements. Different languages has different paradigm. For example C# started with static/strongly type paragoge&#160; but over years it has been evolved and now it does support functional programming as well as dynamic. There are many other languages like F#, Visual Basic and fortran. 
  

It is compiler responsibility to generate assembly with proper intermediate language so that CLR (Common Language Runtime) would be able to generate native code (using JIT) and run at run time. Each compilers support subset if intermediate language and provide its flavour using syntactic sugars in the language.
  

![.Net Languages](http://ahmadrezaa.files.wordpress.com/2014/01/netlanguages_thumb.png ".Net Languages") 
  


  

CLR allows language integration as long as written code comply with Common Language Specification. It means because of differences between languages, if you want to use types generated by other languages, you need to use certain feature of language that are guaranteed to be available in other languages. CLS is the lowest common denominator of language features provided by Microsoft.
  

However, today I don't want to highlight CLS, instead I want to mentions that Intermediate Language in some cases has more features which languages like C# didn't bother to implement. 
  

For example we all know about access modifiers in C#. There is public, private, internal and protected
  <table cellspacing="0" cellpadding="2" width="522" border="2"><tbody>     <tr>       <td valign="top" width="126">Access Modifiers</td>        <td valign="top" width="392">Description</td>     </tr>      <tr>       <td valign="top" width="129">public</td>        <td valign="top" width="389">*public access is the most permissive access level. There is no restrictions on accessing public members*</td>     </tr>      <tr>       <td valign="top" width="132">private</td>        <td valign="top" width="387">*private access is the least permissive access level, private members are accessible only within the body of the class or the struct in which they are declared*</td>     </tr>      <tr>       <td valign="top" width="134">internal</td>        <td valign="top" width="386">*internal types or members are accessible only within files in the same assembly*</td>     </tr>      <tr>       <td valign="top" width="135">protected</td>        <td valign="top" width="385">*A protected member is accessible within its class and by derived class instances*</td>     </tr>      <tr>       <td valign="top" width="136">protected internal</td>        <td valign="top" width="384">visible to derived classes and those of the same assembly</td>     </tr>   </tbody></table>  

&#160;
  

OK, now lets have a look on access modifiers in IL
  <table cellspacing="0" cellpadding="2" width="520" border="2"><tbody>     <tr>       <td valign="top" width="136">IL Access Modifier</td>        <td valign="top" width="250">Description</td>        <td valign="top" width="130">C# equivalent</td>     </tr>      <tr>       <td valign="top" width="138">Private</td>        <td valign="top" width="248">The member is accessible only by other members in the same class or struct</td>        <td valign="top" width="131">private</td>     </tr>      <tr>       <td valign="top" width="139">Family</td>        <td valign="top" width="247">The member is accessible by derived types.</td>        <td valign="top" width="131">protected</td>     </tr>      <tr>       <td valign="top" width="140">famandassem</td>        <td valign="top" width="246">The member is accessible by derived types, but only if the derived type is defined in the same assembly.</td>        <td valign="top" width="131">N/A</td>     </tr>      <tr>       <td valign="top" width="141">Assembly</td>        <td valign="top" width="246">The member is accessible by any code in the same assembly</td>        <td valign="top" width="131">internal</td>     </tr>      <tr>       <td valign="top" width="141">famorassem</td>        <td valign="top" width="246">The member is accessible by derived types in any assembly and also by any types within same assembly</td>        <td valign="top" width="131">protected internal</td>     </tr>      <tr>       <td valign="top" width="141">Public</td>        <td valign="top" width="246">The member is accessible by any code in any assembly</td>        <td valign="top" width="132">public</td>     </tr>   </tbody></table>  

&#160;
  

As you might have noticed there is no C# equivalent for family or assembly although IL has this keyword. there is not such a thing in VB.Net as well. Now lets have a closer look on a simple sample and generated IL code.
  

Assume we have two simple project in a solution. A Console Application and a Class Library which console application referenced that. In the Class Library we have a simple class called Super and in console application, beside Program class we have a class derived from Super class called Sub. Lets have a look on their codes.
  

&#160;
  

Super class source code

```C#

using System;

namespace ClassLibrary
{
    public class Super
    {
        protected internal void Foo()
        {
            Console.WriteLine("Foo");
        }

        internal void Method()
        {
            
        }
    }
}

```


Other class in ClassLibrary Project





``` C#

namespace ClassLibrary
{
    public class Other
    {
        public void Test()
        {
            var super = new Super();
            super.Foo();
        }
        
    }
}
```



Sub class in ConsoleApplication


``` C#

using System;
using ClassLibrary;

namespace ConsoleApplication
{
    public class Sub : Super
    {
        public void Bar()
        {
            Foo();
            Console.WriteLine("Bar");
        }
    }
}

```


Program.cs class in ConsoleApplication


``` C#

using System;

namespace ConsoleApplication
{
    class Program
    {
        static void Main()
        {
            var sub = new Sub();
            sub.Bar();
            Console.ReadLine();
            
        }
    }
}

```



Sub class simply calls Foo method which is defined as “protected internal” which in C# means family *or *assembly in IL. If you build and run application you’ll see that Sub class successfully accesses because although it is not in the same assembly but But sub is actually derived from Super class. the output will be 


```

Foo 
Bar

```


Cool, now let mess around with the IL and see that would happen if we change that.




If you want to dump the IL of an assembly you can simply open a Visual Studio Command and then go to the folder your assembly exist and run ildasm with following parameters


```

ildasm ConsoleApplication.exe /out:ConsoleApplication.il

ildasm ClassLibrary.dll /out:ClassLibrary.il

```


Now, you can open ClassLibrary.il and have a look into generated code. Actually it is a bit long so I just copied part that I’m interested in which is Foo method 



```

  .method famandassem hidebysig instance void 
          Foo() cil managed
  {
    // Code size       13 (0xd)
    .maxstack  8
    IL_0000:  nop
    IL_0001:  ldstr      &quot;Foo&quot;
    IL_0006:  call       void [mscorlib]System.Console::WriteLine(string)
    IL_000b:  nop
    IL_000c:  ret
  } // end of method Super::Foo

```


You see that protected internal compiled to famorassem. What would happen if we change this to famandassem and assemble this IL again? Answer is CLR will prevent calling this method and will raise MethodAccessException exception when we call this from console application. But let put this into action and actually test this. So, change the famorassem to famandassem and save the file and use ilasm.exe from Visual Studio Command to assemble this code again. Before assembling this you can delete current ClassLibrary.Dll. To assemble use following command parameters:




```

ilasm ClassLibrary.il /out:ClassLibrary.dll /dll

```


If you get Operation completed successfully at the end new class library is generated. If you run ConsoleApplication.exe you’ll get following exception because of violating the access level defined in the IL





```

Unhandled Exception: System.MethodAccessException: Attempt by method 'ConsoleApp
lication.Sub.Bar()' to access method 'ClassLibrary.Super.Foo()' failed.
   at ConsoleApplication.Sub.Bar() in f:\Users\ara\Documents\Visual Studio 2013\
Projects\ConsoleApplication3\ConsoleApplication3\Sub.cs:line 10
   at ConsoleApplication.Program.Main() in f:\Users\ara\Documents\Visual Studio
2013\Projects\ConsoleApplication3\ConsoleApplication3\Program.cs:line 10

```






famandassem means that only derived class within same assembly will have access to that member. Just for the sake of testing lets Super to the ConsoleApplication and see if it works.




Open ClassLIbrary.il in a text editor. Copy following IL code which is Super class il code


```

// =============== CLASS MEMBERS DECLARATION ===================

.class public auto ansi beforefieldinit ClassLibrary.Other
       extends [mscorlib]System.Object
{
  .method public hidebysig instance void 
          Test() cil managed
  {
    // Code size       15 (0xf)
    .maxstack  1
    .locals init ([0] class ClassLibrary.Super super)
    IL_0000:  nop
    IL_0001:  newobj     instance void ClassLibrary.Super::.ctor()
    IL_0006:  stloc.0
    IL_0007:  ldloc.0
    IL_0008:  callvirt   instance void ClassLibrary.Super::Foo()
    IL_000d:  nop
    IL_000e:  ret
  } // end of method Other::Test

  .method public hidebysig specialname rtspecialname 
          instance void  .ctor() cil managed
  {
    // Code size       7 (0x7)
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call       instance void [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  } // end of method Other::.ctor

} // end of class ClassLibrary.Other

.class public auto ansi beforefieldinit ClassLibrary.Super
       extends [mscorlib]System.Object
{
  .method famandassem hidebysig instance void 
          Foo() cil managed
  {
    // Code size       13 (0xd)
    .maxstack  8
    IL_0000:  nop
    IL_0001:  ldstr      "Foo"
    IL_0006:  call       void [mscorlib]System.Console::WriteLine(string)
    IL_000b:  nop
    IL_000c:  ret
  } // end of method Super::Foo

  .method assembly hidebysig instance void 
          Method() cil managed
  {
    // Code size       2 (0x2)
    .maxstack  8
    IL_0000:  nop
    IL_0001:  ret
  } // end of method Super::Method

  .method public hidebysig specialname rtspecialname 
          instance void  .ctor() cil managed
  {
    // Code size       7 (0x7)
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call       instance void [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  } // end of method Super::.ctor

} // end of class ClassLibrary.Super


// =============================================================

```


Now open ConsoleApplication.il and paste this code at the end after end of Class ConsoleApplication.Sub. 




We need to do one more thing and get rid off reference to ClassLibrary.il which we don't need anymore. So right in the beginning there is some code for that. Delete that piece of code and save the file.


```

.assembly extern ClassLibrary
{
  .ver 1:0:0:0
}

```


One more thing to do and it is replacing all [ClassLibrary] with empty string.




Delete console application and assemble the il to make new ConsoleApplication.exe using following command


```

ilasm ConsoleApplication.il /exe /out:ConsoleApplication.exe

```


Now we have new ConsoleApplication which doesn't have reference to ClassLibrary. Run ConsoleApplication.exe you see it runs successfully.

