Download Link: https://assignmentchef.com/product/solved-want-to-play-a-game
<br>
<h1>Introduction</h1>

This is a simple exercise to compile and insert a small program into the kernel i.e. a kernel module. The program that you will write doesn’t do anything useful but you will learn the basics steps of writing a kernel module.

You will need root privileges on the machine that you are working on. The safest way is of course if you run everything using a virtual machine but you could of course use your regular machine (at your own risk). You will not be able to use the KTH computers since you don’t have root access to those and will (at least as it is now) not be able to start up a virtual machine on those.

The kernel is of course the heart of the operating system and has complete access to the hardware that it is running on. User space processes are protected from each other but the kernel has complete access to everything. If we do a mistake in our user space, a process will crash but the kernel survives – a mistake in the kernel will most often result in that the machine crashes (but you will most often be able to restart it).

There are basically two ways of extending the kernel, either we compile our own modi ed kernel or we insert a module into a running kernel. The latter approach is of course much more convenient but it requires that we know what we’re doing (or at least have an idea of what we’re doing).

<h1>1           A     rst try</h1>

The program that we will write is a regular C program but with some special properties. It does not contain a main procedure, instead it will have several procedures that will be used by the kernel, and could be made available to user level programs.

The two most important procedures are the ones used when the module is loaded and removed from the kernel. Save the following in a le called hal.c.

#include &lt;linux /module .h&gt; #include &lt;linux / kernel .h&gt; #include &lt;linux / i n i t .h&gt;

MODULE_LICENSE(“GPL” );

MODULE_AUTHOR(“Dr Chandra” );

MODULE_DESCRIPTION(” Heuristically                              programmed ALgorithmic                  computer” );

static            int        __init            hal_init (void)

{

printk (KERN_INFO “I             honestly        think        you ought      to        calm down;
” );

return       0;

}

static                      void __exit hal_cleanup (void)

{

printk (KERN_INFO “What are              you doing ,         Dave?
” );

}

module_init ( hal_init ); module_exit ( hal_cleanup );

In this program we rst include some header les that we need. We then describe our kernel module and it is important that we set thelicense to GPL i.e. that this is free software and not some proprietary code. We also provide two straneg looking functions that we use in the two last macros: module_init() and module_exit(). Our kernel module will do nothing but write out a message when it is inserted into the kernel and when it is removed from the kernel.

We should now compile this but we need to compile it using some special libraries that are used when the kernel is built. If you look in the directory /lib/modules, you will nd a directory for each linux kernel that you have installed on your computer (or virtual machine). In these directories you will nd a link called build that is referring to the source directory of the kernel.

<h2>1.1           a Make le</h2>

The source directory has a Makefile that will do the compilation for us. To make things easy, we create our own make le in our source directory that will do the making for us. Create a le called Makefile in the directory that holds the le hal.c. The trick with the uname -r will get us the number of the running kernel and thus direct us to the right directory.

obj−m += hal . o

<table width="648">

 <tbody>

  <tr>

   <td width="319">a l l :</td>

   <td width="225"> </td>

   <td width="45"> </td>

   <td width="58"> </td>

  </tr>

  <tr>

   <td width="319">make −C / lib /modules/$( shellclean :</td>

   <td width="225">uname −r )/ build M=$( shell</td>

   <td width="45">pwd)</td>

   <td width="58">modules</td>

  </tr>

  <tr>

   <td width="319">make −C / lib /modules/$( shell</td>

   <td width="225">uname −r )/ build M=$( shell</td>

   <td width="45">pwd)</td>

   <td width="58">clean</td>

  </tr>

 </tbody>

</table>

Now if everything works you should be able to make the module from the directory of the make le.

$ make :

: make[1]: Leaving directory ’/usr/src/linux-headers-4.4.0-34-generic’

Now take a look in the directory and you will nd tons of new les. If you use the command ls -a you will see even more. The one that we are interested in is the le hal.ko, this is the kernel module object le.

<h2>1.2           loading the module</h2>

So we have an object le and the only thing now is to load it into the kernel. You can take a look at all the modules that are already loaded into the kernel by using the lsmod command.

$ lsmod :

:

Now if you have the right privilege you should be able to insert also our module. Try this:

$ sudo insmod hal.ko

If nothing happens you have probably succeeded. Now issue the command dmesg that will print a le containing all messages from the kernel since you booted your machine. Pipe the output to tail to see the last rows. The last entry is hopefully a message from our HAL module – it’s now in control of your machine.

$ dmesg | tail

:

:

You can also verify, using lsmod, that the module is in fact among the loaded modules. To remove it from the kernel we use the command rmmod giving the name of the module as an argument.

$ sudo rmmod hal

To show you that the module is actually loaded in to kernel space we can check the address it is loaded to. Change the print-out when the module is loaded to the following, make, load the module and check the message from dmesg. Is it an address in kernel space?

here :

printk (KERN_INFO “I ’m here %p ); n” , &amp;&amp;here );

<h1>2           Now what</h1>

This assignment is only scratching on the surface of kernel space programming. Moving forward from here things require a lot more coding; it’s not complicated but it’s more code. One would like to simply add a procedure in the loaded kernel module and then make this procedure accessible as a system call. This would make things easy for the programmer but probably result in a bowl of spaghetti once everyone added their own system calls. The number of true system calls should be kept small to make the operating system easier to manage and control.

The alternative way of communicating with our kernel module is to let it show up with a regular le interface. To the user level programs it looks like a le but under the hood the module is doing the work.

<h2>2.1           Dr Dyson to your help</h2>

Create a new directory called skynet and copy the make le we used to the new directory. Edit the Make le so that it now used an object le called skynet.o . Create a le called skynet.c with the following code; we will go

through the code so that you know why it is there.

<table>

 <tbody>

  <tr>

   <td width="72"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

The header      les are as before but we are now going to add a le system interface to the module. We therefore include two more     les proc_fs.h and seq_file.h. The macros that describe the module are required so we write some fun things.

#include &lt;linux /module .h&gt;                              //       included       for     a l l      kernel       modules

#include &lt;linux / kernel .h&gt;                                //       included          for KERN_INFO

#include &lt;linux / i n i t .h&gt;                                   //       included       for                        __init and __exit macros

#include &lt;linux /proc_fs .h&gt; // f i l e operations #include &lt;linux / seq_file .h&gt; // seq_read , . . .

MODULE_LICENSE(“GPL” );

MODULE_AUTHOR(“Dr.           Dyson” );

MODULE_DESCRIPTION(“Global                    Information         Grid” );

Next, we are de ning a file_operations data structure. This structure is populated with call-back functions that the kernel will use when the le interface of the module is used. The kernel needs to know how the le should be opened, closed, read from etc. The only function that we will provide is the function used to open the le skynet_open().

<table width="551">

 <tbody>

  <tr>

   <td width="60">static</td>

   <td width="34">int</td>

   <td width="166">skynet_show( struct</td>

   <td width="112">seq_file ∗m,</td>

   <td width="121">void ∗v );</td>

   <td width="57"> </td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="34">int</td>

   <td width="166">skynet_open ( struct</td>

   <td width="112">inode ∗inode ,</td>

   <td width="121">struct                   f i l e</td>

   <td width="57">∗ f i l e );</td>

  </tr>

 </tbody>

</table>




static                const struct               file_operations                skynet_fops = {

. owner = THIS_MODULE,

. open = skynet_open , . read = seq_read ,

. llseek = seq_lseek ,

. release = single_release ,

};

static    int           skynet_show( struct       seq_file ∗m,       void ∗v) { here :

seq_printf (m,          “Skynet         location :         0x%lx 
” ,

(unsigned long)&amp;&amp;here );

return      0;

}

static            int          skynet_open ( struct             inode ∗inode ,          struct                  f i l e ∗ f i l e ) {

return          single_open ( f i l e ,             skynet_show , NULL);

}

We then provide the functions that will be used when the module is loaded and unloaded. This time we will actually do some work here. We will register the module as a proc module and provide a name skynet that will show up the the /proc directory. Wen the module is unloaded we remove the proc-entry.

static            int        __init               skynet_init (void) {

proc_create (“skynet” , 0 , NULL, &amp;skynet_fops ); printk (KERN_INFO “Skynet in control 
” );

return      0;

}

static                         void __exit skynet_cleanup (void) {

remove_proc_entry(“skynet” , NULL);

printk (KERN_INFO “I ’ l l               be back !
” );

}

module_init ( skynet_init ); module_exit ( skynet_cleanup );

<h2>2.2           in control</h2>

Make the module and load it into the kernel, check the output from dmesg to make sure that it was loaded ok. Then take a look in the /proc directory, do we have a skynet le? Take a closer look at the le using ls -l, what does it look like, how big is it?

$ ls -l /proc/skynet :

Now try to read the        le using the cat command.

$ cat /proc/skynet :

This is how all the les you see in the /proc directory work, they are simply interfaces to di erent kernel services.

<h1>3           Device drivers</h1>

Take a look in the directory /dev, all of the les that you nd there are also interfaces to kernel services. If you use the command ls -l you will see that the rst letter of each description is either d, l or something that you might not have seen before: b or c. The latter les are block and character devices.

We will now try to add a kernel module that shows up as a character device.

<h2>3.1           Joshua</h2>

There will be some code but you will manage. Create a new directory called joshua, and in there a new source code le joshua.c and a header le joshua.h. Also make a copy of the Make le and place it in the directory. Edit the Make le so that it will use the object le joshua.o. Now for the code, we will start with the header le.

#ifndef JOSHUA

#define JOSHUA

#include &lt;linux / i o c t l .h&gt;

// This             is     the       required        size      of     the       buffer .

#define JOSHUA_MAX 40

// This macro               w i l l     give      us    the      right      i o c t l     code .

#define JOSHUA_GET_QUOTE _IOR(0 xff ,                           1 , char ∗)

#endif

We will create a kernel module that will return quotes by Joshua. We have a header le that de nes two macros JOSHUA_MAX and JOSHUA_GET_QUOTE.

The rst is the longest string that a quota can be (including trailing zero). The user level program should allocate a bu er with a least this size and pass a pointer to the bu er to the joshua module. The joshua module will then copy a new quote from Joshua into the bu er.

The second macro is a bit cryptic but it uses a macro from the ioctl.h header le that helps us create a hopefully unique ioctl number. The number is created from one magic number (0xff), a number that is unique for the joshua module and the data type that we will pass from the user to the kernel module. We have here chosen 0xff as our magic number but some more care should go in to this if we actually did something serious.

<h2>3.2           the user program</h2>

The user program that should request quotes from joshua could look like follows. Create a source le called quote.c in the same directory as joshua (not required but we need to nd the joshua header le so we might as well place it there).

#include &lt;stdlib .h&gt;

#include &lt;stdio .h&gt; #include &lt;fcntl .h&gt;

#include &lt;unistd .h&gt; #include &lt;string .h&gt;

#include &lt;sys/ i o c t l .h&gt; #include &lt;sys/types .h&gt;

#include “joshua . h”

int main ()                {

char ∗file_name = “/dev/joshua” ; int fd ; fd = open( file_name , O_RDONLY);

if          ( fd == −1) {

perror (“Joshua         is     not         available ” );

return       2;

} char buffer [JOSHUA_MAX] ; if ( i o c t l ( fd , JOSHUA_GET_QUOTE, &amp;buffer ) == −1) { perror (“Hmm,         not         so           good” );

} else { printf (“Quote − %s
” , buffer );

<table>

 <tbody>

  <tr>

   <td width="72"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

} close            ( fd );

<table width="560">

 <tbody>

  <tr>

   <td width="74">return}</td>

   <td width="487">0;The user program opens a le, /dev/joshua, in read mode and then makes a system call ioctl() that will make the magical request to the joshua module. Not that nothing is working so far, we have not created our module nor made it accessible from the device le. This is just to get an idea of how things will work in the end.As you see the program creates a bu er on the stack and passes the address of this bu er to the system call. This is why we said that the data type was char* when we de ned the ioctl number.3.3         the moduleSo now we are ready to de ne our kernel module joshua.c. This will have the same components as the hal module that we created before but will also register itself using the ioctl functionality. We will go through the code part by part and explain why it is there.The rst part is a sequence of included header les. Some you have seen from hal.c and skynet.c but many are new. It’s not important to keep track of which les are needed since this is quite easily determined. We also include our own joshua.h so we use the same size of the bu er and the same ioctl number as the user space program.</td>

  </tr>

  <tr>

   <td colspan="2" width="560">#include &lt;linux /module .h&gt;        //            a l l         kernel   modules #include &lt;linux / kernel .h&gt;            // KERN_INFO#include &lt;linux / fs .h&gt;                                         //          file_operations           . .#include &lt;linux /cdev .h&gt;                                    //        cdev_init ,           cdev_all        . . .#include &lt;linux / device .h&gt;                                //         class_create         . . .#include &lt;linux /uaccess .h&gt;                                  // copy_to_user#include “joshua . h”                                             //     to       agree on      the        interface</td>

  </tr>

 </tbody>

</table>

As before we need to describe the module.

MODULE_LICENSE(“GPL” );

MODULE_AUTHOR(“Prof             Franken” );

MODULE_DESCRIPTION(“Joshua” );




We then de ne some macros and data structures that are needed when we de ne our device driver. The important thing here is the le operations data structure joshua_fops. As before we populate it with the functions that the kernel needs but now we include a function called joshua_ioctl that will be the interface to the device driver.

#define FIRST_MINOR 0

#define MINOR_CNT 1

<table width="534">

 <tbody>

  <tr>

   <td width="60">static</td>

   <td width="474">int            joshua_open ( struct            inode ∗i ,          struct         f i l e ∗ f );</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">int             joshua_close ( struct             inode ∗i ,          struct         f i l e ∗ f );</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">long                                           joshua_ioctl ( struct    f i l e ∗f , unsigned int cmd,            unsigned long arg );</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">dev_t dev ;</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">struct            cdev c_dev ;</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">struct             class ∗ cl ;</td>

  </tr>

  <tr>

   <td width="60">static</td>

   <td width="474">struct               file_operations                joshua_fops = {</td>

  </tr>

 </tbody>

</table>

. owner = THIS_MODULE,

. open = joshua_open ,

. release = joshua_close ,

. unlocked_ioctl = joshua_ioctl

};

Next follows declarations of our own data structures. We de ne an array of three quotes and an integer that we will increment to keep track of the next quote to deliver.

#define QUOTES 3

static                      const char ∗ quotes [QUOTES] = {

“Why play a game that                cannot be won?” ,

“Mutual         destruction         is       not a        victory . ” ,

“Simulation         is     the     mother     of      knowledge . ”

};

static            int         next = 0;

Now for the basic de nitions of the kernel module i.e. what should be done when the module is loaded and unloaded. This is of course where we want to register the module under a device name i.e. /dev/joshua. The rst one is the procedure when the module is loaded. It looks quite scary but most of it is code to handle things that could go wrong. If you go through the code you will see that it basically, does ve things:

<ul>

 <li>alloc_chrdev_region(): allocates a device number dev</li>

 <li>cdev_init(): initializes the character device structure c_dev</li>

 <li>cdev_add(): adds the character device given the device number</li>

 <li>class_create(): creation of a device class called char</li>

 <li>device_create(): creating the device given the class, device number an name joshua</li>

</ul>

Don’t ask me how things work, the important thing is that it is doable. We can worry about what actually is happening later. Notice that most of the code is checking if the operations were successful and if not undoing the previous operations before returning an error message.

static            int        __init               joshua_init (void) {

int      ret ;

struct device ∗dev_ret ; printk (KERN_INFO “Want to play a game?
” );

if                       (( ret = alloc_chrdev_region(&amp;dev , FIRST_MINOR, MINOR_CNT,                                      “joshua” )) &lt; 0) {

return       ret ;

} cdev_init(&amp;c_dev , &amp;joshua_fops );

if             (( ret = cdev_add(&amp;c_dev ,                    dev , MINOR_CNT)) &lt; 0) {

return       ret ;

}

if                  (IS_ERR( cl = class_create (THIS_MODULE,                    “char” )))          {

cdev_del(&amp;c_dev ); unregister_chrdev_region (dev , MINOR_CNT);

return PTR_ERR( cl );

}

if                    (IS_ERR( dev_ret = device_create ( cl , NULL,                      dev , NULL,           “joshua” )))        {

class_destroy ( cl );

cdev_del(&amp;c_dev ); unregister_chrdev_region (dev , MINOR_CNT);

return PTR_ERR( dev_ret );

}

return      0;

}

When the module is unloaded we must remove everything that we have created. We destroy the device, the class and the cdev structure – basically, undoing everything we did when the module is loaded. We also add a print statement so we know that the module was successfully unloaded.

static                void __exit                joshua_exit (void) {

device_destroy ( cl , dev ); class_destroy ( cl );

cdev_del(&amp;c_dev );

unregister_chrdev_region (dev , MINOR_CNT);

printk (KERN_INFO “How about a                nice game of           chess ?
” );

}

We will now de ne what should be done when someone opens or closes the le . For the user level program the device looks like a le and the obvious operations are thus open, close etc. As you see below, we don’t do anything special when someone tries to open or close the le but here is where we could initialize or deleted data structures that pertains to the session.

static            int          joshua_open ( struct            inode ∗i ,          struct          f i l e ∗ f ) {

return      0;

}

static            int            joshua_close ( struct             inode ∗i ,          struct          f i l e ∗ f ) {

return      0;

<table width="666">

 <tbody>

  <tr>

   <td width="61">}</td>

   <td width="573">Now for the heart of our module, the things we will do when someone issues a ioctl() command. This is where we will return a quote from Joshua. Remember that the client allocated a bu er and sent us a address to this bu er. It’s our job to copy one of the quotes that we have into this bu er.As you see below the ioctl procedure takes three arguments: a le descriptor, a command and the argument that was passed from the user. The command is a number that has been generated by the _IOR macro. The user program implicitly used this when using the JOSHUA_GET_QUOTE macro in the joshua.h header le. We can now use the same macro and have a switch statement that looks at cmd and hopefully jumps to the right case.You now see how we very easily can add new commands to our module.</td>

   <td width="32"> </td>

  </tr>

  <tr>

   <td width="61">static {</td>

   <td width="573">long             joshua_ioctl ( struct              f i l e ∗f ,             unsigned int cmd,                       unsigned long</td>

   <td width="32">arg )</td>

  </tr>

 </tbody>

</table>

switch (cmd)               { case JOSHUA_GET_QUOTE:

next = ( next+1) % QUOTES;

printk (KERN_INFO “Joshua :            copy     to      buffer         at 0x%lx 
” ,         arg );

if             (copy_to_user (( char ∗) arg ,                  quotes [ next ] , JOSHUA_MAX))

{ return −EACCES;

} break ;

default :

return −EINVAL;

}

return      0;

}

To handle a JOSHUA_GET_QUOTE request, we increment the next variable, print a message and copy the next quote into the bu er provided to us in arg. Note that this is a very scary procedure in that we don’t really know what the user sent us. In the best case it is actually a memory reference to a bu er that is at least JOSHUA_MAX large. If the user made mistake, the bu er is too small or the address is pointing randomly somewhere else. In the worst case the user is luring us into writing something to a memory segment that belongs to the kernel. The user has of course no possibility to write to these segments but the joshua module belongs to the kernel and has complete access to the kernel space. This is why we use the procedure copy_to_user() that will check that the address actually belongs to the user address space and only then copy the string.

The last thing we do is use the macros module_init() and module_exit() to register our procedures that should be used when the module is loaded and unloaded.

module_init ( joshua_init ); module_exit ( joshua_exit );

This is it, you should be set to go.

<h1>4           Want to play a game</h1>

Compile the joshua module using the make le in the joshua directory. Then load the module using insmod.

$ make :

:

$ sudo insmod joshua.ko

Now take a look in the /dev and you should see a joshua device. Do ls -l and you will have more information.

$ ls -l /dev/joshua crw——- 1 root root 244, 0 aug 19 10:16 /dev/joshua

Hmm, a character device (the ’c’ in the beginning) with read and write privileges for root, who is also the owner of this le. To be able to open it from a regular user we need to change the privilege level.

$ sudo chmod o+r /dev/joshua

Do ls -l again and see that the mode of the device now allows “other” users to read the device. Also take a look in /sys/devices/virtual/char and you will see a directory containing information about our device. We don’t really need to know what is going on here but I’m quite sure there is some magic involved. The important thing is that we have our device available for the users.

Now switch attention to the program quote.c, compile it and run a test, does it work?

<h1>5           Summary</h1>

This assignment is of course only scratching the surface of kernel modules or device driver implementation. The important lesson is that it is fairly easy to add things to the kernel and that user level programs can interact with kernel module using the device interface.

You’re encouraged to play around some more, add more commands or ponder what would happen if two processes call the module at the same time. When you experiment, keep in mind that you’re playing with the kernel; hopefully on a virtual machine but if you’re like me you of course run it on your own laptop where you also have all you non-backed photos. Another thing you must know is that most of the libraries that you are used to use are not available to the kernel. The libraries are written to be called from user space and maybe trap to the kernel for a system call – but we are already in the kernel. This is why the code above have used printk for output.

<h1>Appendix</h1>

Hear are some comments on things that could go wrong and how to      x them:

<h2>things do not work</h2>

Do you have make installed? If not install it using sudo apt install make . You will use make in more assignments so it’s a good thing to have anyway.

You need to have the library development les i.e. the header les that we refer to. If things go wrong when you make the modules this could be the problem. Install them using sudo apt install libelf-dev

Make sure that the make           le is actually called Make le . When you run make it will look for this     le in the current directory. You could change this using the -f       ag but we will only use one Make le for each experiment and each experiment has its own directory.

The Make le must of course be adapted for the di erent experiments. If the rst Make le is used for hal.o then you will have to change this to for example skynet.o .

<h2>what is going on</h2>

The Make le de nes a variable obj-m += hal.o (or rather adds hal.o to

the obj-m variable. This variable is used by the Make le in the lib

modules

.. directory. It will be the object le that is turned in to a kernel object le. When we run make, it will read our Make le and then, if no arguments are given, perform what is listed under all: . What we say there is that it should run make but now with some arguments.

The rst argument to make is the -C ag together with a mysteriously looking directory. The ag will tell make to go to this directory and then do what ever needs to be done there. On my current machine the directory is:

/lib/modules/4.15.0-36-generic/build

but this will of course change when ever I upgrade my system. To avoid having to change the Make le every time I use the trick of calling the command uname . Try the command uname -a in a shell and you will see a lot of information about your system. The thing that we are looking for is the kernel revision that we get from uname -r .

In the same way we de ne the variable M to be equal to the directory where we are currently standing. This is where the source les are found and this is were the results should be written. We also give the argument modules since we want make to generate the kernel modules for us.