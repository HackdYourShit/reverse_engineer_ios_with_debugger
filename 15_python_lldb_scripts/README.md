## Using Python to write LLDB scripts
### Basics
```
(lldb) script print lldb.target
SampleApp-Swift

(lldb) script print lldb.thread
thread #1: tid = 0x67f3e, 0x0000000108005244 SampleApp-Swift`main at AppDelegate.swift:14, queue = 'com.apple.main-thread', stop reason = breakpoint 1.4

(lldb) script print lldb.process
SBProcess: pid = 16218, state = stopped, threads = 1, executable = SampleApp-Swift

(lldb) print lldb.frame.registers

(lldb) script lldb.debugger.HandleCommand("frame info")
```
### SBValue is a friend
```
>>> reg = lldb.frame.FindRegister("arg3")

>>> print(reg)
(unsigned long) rdx = 0x00007ffeefbff598

>>> print(reg.value)
0x00007ffeefbff598

>>> print(reg.description)
<__NSStackBlock__: 0x7000053adba8>
... long description of Blocks of code.
... another long description of Blocks of code.
```
### Print the correct thing
```
>>> print(lldb.SBFrame)
<class 'lldb.SBFrame'>

>>> lldb.frame
<lldb.SBFrame; proxy of <Swig Object of type 'lldb::SBFrame *' at 0x114bdc330> >

>>> print(lldb.frame)
frame #0: 0x00000001000013df objc_play`main at main.m:26:42
```
### Help
```
(lldb) script help(lldb.process)

(lldb) script help(lldb.frame)

(lldb) script help(lldb.SBDebugger)

>>> help(lldb.debugger.GetSelectedTarget)

```
### Script interface
```
(lldb) script
>>> 2+3
5

>>> print lldb.debugger.GetVersionString()
lldb-902.0.79.2
Swift-4.1

>>> print lldb.debugger.GetSelectedTarget()
SampleApp-Swift
```
### Print Register value
```
>>> reg = lldb.frame.FindRegister("arg3")
>>> print(reg)
(unsigned long) rdx = 0x00007ffeefbff598
>>> print(reg.GetValue())
0x00007ffeefbff598
```


### Well placed Breakpoint
```
>>> print lldb.frame.GetFunctionName()
tinyDormant.YDJediVC.viewDidLoad() -> ()


>>> print lldb.frame.get_all_variables()

(tinyDormant.YDJediVC) self = 0x00007fc2421187b0 {
  UIKit.UIViewController = {
    UIKit.UIResponder = {
      ObjectiveC.NSObject = {}
    }
  }
  secret_number = (_value = 10)
  secret_string = "foobar"
  secret_nsstring = 0xfe1420463360b96a "ewok"
}

```

### Instantiate and read ObjC Class
```
>>> options.SetLanguage(lldb.eLanguageTypeObjC)


// you don't need print, but you won't get usable feedback if you don't use it
>>> print lldb.frame.EvaluateExpression('[MispelledByDesign new]')
 = <could not resolve type>

>>> a = lldb.frame.EvaluateExpression('[UIViewController new]')

>>> print a
(UIViewController *) $1 = 0x00007fab33c12140

>>> print a.description
<UIViewController: 0x7fab33c12140>
```
### Instantiate and read a Swift Class
```
(lldb) script
>>> options = lldb.SBExpressionOptions()
>>> options.SetLanguage(lldb.eLanguageTypeSwift)
>>> options.SetCoerceResultToId()
>>> e -lswift -O -- ASwiftClass()
error: <EXPR>:3:1: error: use of unresolved identifier 'ASwiftClass'
ASwiftClass()
^~~~~~~~~~~

(lldb) e -lswift -- import Allocator
(lldb) e -lswift -O -- ASwiftClass()
<ASwiftClass: 0x60000029cd40>

>>> b = lldb.target.EvaluateExpression('ASwiftClass()', options)
>>> print b.GetValueForExpressionPath('firstName')
No value

>>> script print b.GetValueForExpressionPath('.firstName')
firstName = "Carlos"

(lldb) script print b
(Allocator.ASwiftClass) $R1 = 0x00006040002918a0 {
eyeColor = 0x000060000066f040 {
ObjectiveC.NSObject = {}
}
firstName = "Carlos"
lastName = "Jackel"
}

>>> mynumber = lldb.frame.FindVariable("i")

>>> mynumber.GetValue()
'42'
```

### Troubleshooting
If you hit `unable to execute script function` make sure your `madman.py` filename matches the name in this code:
```
debugger.HandleCommand('command script add -f madman.GetBundleIdentifier yd_bundle_id')
```
### References
https://lldb.llvm.org/use/python-reference.html

https://github.com/theocalmes/lldbpy

http://ryanipete.com/blog/lldb/python/how_to_create_a_custom_lldb_command_pt_1/

https://lldb.llvm.org/use/python.html

https://aijaz.net/2017/01/11/lldb-python/index.html

https://hub.packtpub.com/lldb-and-command-line/
