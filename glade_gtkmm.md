#Compile .glade file using glib-compile-resource' and include it in my C++ project
I have found so many different tutorials on how to use Glade with gtkmm (Gtk+ 3). 
The best example so far is the official Gnome developer guide, but it uses the
XML file and not a compiled version.
I do not want to use a XML file, as I want it to be a single file executable.

There are two ways to do this:
1) Just wrap the entire XML file as a string variable
2) Use Resource loader from compiled UI file.

I want #2 to work.

##First go
Here is an example using the XML file:




