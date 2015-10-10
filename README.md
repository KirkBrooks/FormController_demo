# FormController_demo
This demo shows how to use the idea of a form controller in 4D forms. 

4D allows you to attach code to any object on a form as well as to the form itself. It's been this way since the first version and that ability was a pretty big deal thirty years ago. It's still vital you have the ability to individually control form objects, of course, but it can quickly become challenging if you have dozens of small methods residing in various objects scattered around a very complex form. Tracking the way the code on these objects interacts and udpates the form can be a nightmare. 

The idea of a form controller is to move most or all of that code to a single method that can be easily called by any form object. Having all the code in one place greatly simplifies understanding and maintaining a complex form. 


##The big idea
The idea of a form controller has been tried by some devs in the past. With 4D's embrace of form objects in v13 it became possible to reference an object from the code running on the form even if you didn't expressly pass some reference to it. The key to this is the **OBJECT Get name** function. Any piece of code you put in an object can call that function and get the name of the object that ran it. So with that it's possible to do this:

```
//  Contact_form_controller
C_TEXT($1;$2;$action;$errMsg)

If (Count parameters=0)
	$action:=OBJECT Get name(Object current)
Else 
	$action:=$1
End if 
  // ------------------------------------------

Case of 
: ($action="formEvents")
	Case of 
	: (Form event=On Load)
		FORM_INHERIT_SCRIPT ("Contact Form";"hideNav:true,acceptMethod:Contact_form_acceptCallBack")
	End case 
: ($action="name")
	Object_SET_toValue ($action;Uppercase(Object_get_valueText ("name")))
: ($action="phone")
	Object_SET_toValue ($action;Phone_format_string (Object_get_valueText ("phone")))
: ($action="email")
	$errMsg:="The email address is: \r\r  "+Object_get_valueText ("email")
: ($action="btn_button")
	$errMsg:="You clicked the button."
Else 
	$errMsg:="Unkown action or object: "+$action+"\r\rForm Event: "+4D_FormEvent_text +"  {"+Current method name+"}"
End case 

  // ------------------------------------------
4D_ALERT ($errMsg;Current method name)
```
This is the form controller for the Contact dialog in the demo. I can put it in any object on the form and it will call this method which can then determine which object made the call. I can also force a call by including the name of the object as $1. In fact this is how I make it respond to form events. 

So the first thing I do is determine what action I need to take based on what called the method. A case statement let's me list them out. Since this is always being seen by a user I built in an immeadiate feedback system with $errMsg and a custom method **4D_ALERT**. The alert does nothing if $errMsg is empty. This gives me a handy way to alert the user to issues or simply provide information that consistent and easy. 

####Why "$action" instead of "$object"?
Because I see the method as managing actions initiated by the objects. The object made the call but now the method takes some action. That's just my preference. Feel free to change the variable names as you will. 

##On to the demo...
The demo is written in 4D v13.6. This version was recently sunset by 4D but you should be able to open it with newer versions for the forseable future. 

Run **CONTACT_DEMO** first. This is a simple method which opens a little dialog for entering contact information. There are no process variables used - only form variables. It's function is trivial but shows several cool features of 4D's form system:
    Inhereited forms
    Object vars 
    
Now check out the Messages table. This is an actual input form instead of a dialog. Just double click on a record from the list. Notice the familiar 4D vRecNum variable is there too - but I changed two things: I made it a form variable instead of a process variable and I moved the code to the form controller instead of the object. 

##This idea scales well
These are simple examples. I have some very complex forms that run to 400+ lines. That kind of complexity is extremely difficult to manage if you have to go looking into individual button/list/field/variable/listbox/webarea objects to see what that object is doing and how it relates to the rest of the group. Having all the objects refer to the single controller method gets the focus in one place. I can group the various action calls (my term) for each of editing. I can move long processing into other methods and retain a logical representaton of what's going on.

Another nice feature is I can duplicate objects on the form and only need to change the object name to have it work. This makes laying in columns of option buttons really easy. And if I forget to 'wire up' one of them the error pops up when it gets clicked.

Sometimes it's useful to have a single form method for all input forms on a table. I do this with less complicated tables. The form controller can have all sorts of action calls on it but you only reference the ones you need from specific objects. So you can catalog all of the actions that may be taken from a form in the form controller and expose them only whne you make a suitably named object.

##This makes subforms much easier to work with
The promise of subforms is to be able to build truly portable forms that can be dropped anywhere (mostly). This is a little beyond the scope of this demo but a subform with a single form controller like this is much easier to manage.

##Check out the inhereted form 
I've been using this sort of inhereted form for a long time and I like it. I have several versions of varying length for different applications but they all have the same objects and names. They are accessed using the **FORM_INHERIT_SCRIPT** method. At it's simplest you give it the title you want to appear in the bar. Other options available are listed: 
```  
  // Method: FORM_INHERIT_SCRIPT (text{;text})
  // $1 is the text for Sample Text - "" to hide
  // $2 is tuple of params:
  //   hideAccept   - default is false
  //   hideCancel   - default is false
  //   hideNav      - hide the navigation buttons
  //   bgColor      - HEX color for background  #rrggbb
  //   foreColor    - HEX color foreground
  //   acceptMethod - method to run when Accept button is clicked
  //   objName      - name of header object. Allows you to manage more than one header object - if you really want to
  //   helpMethod   - method to run when Help button is clicked
  //                  help button not shown unless this is defined
  ```
  The second parameter takes a very simplified JSONesque string of keys and values. It's all a single string so it's very simple with no other commas, colons, spaces or double quotes allowed. 

To set the title and hide the navigaton buttons: 
  ```
  FORM_INHERIT_SCRIPT("The Title";"hideNave:true") 
```
To set the title, hide the nav buttons and hide the accept button:
  ```
  FORM_INHERIT_SCRIPT("The Title";"hideNave:true,hideAccept:true") 
```
###Callbacks for Accept and the Help button
To run a method when the user clicks the accept button include the methodname as the value for acceptMethod. That method will be the only thing to run when the user clicks the accept button so you can error check, do other actions, etc. The form won't close unless you issue an ACCEPT or CANCEL command in your method. 

If you include a helpMethod the help button is made visible and that method runs when it's clicked. 

##Useful object methods
Here are a copule of methods I find really useful for working with objects on forms. 
- **Object_get_valueText** (object name ; {subform name}) => object's value as text
- **Object_SET_toValue** (object name; text value)  
    Works on process variables too. Converts the text value ($2) to whatever the variable type is. 
- **Object_set_toPointer** (object name; pointer) 
    Will set the object to the value of whatever the pointer references. Converts pointer data to object type. 
  
##Hope this helps
  These techniques helped me very much to improve my 4D forms. I hope you find them helpful too. 
