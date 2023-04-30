Download Link: https://assignmentchef.com/product/solved-hello-dolly
<br>
<h1>                            1        Introduction</h1>

This is an experiment where we will explore how processes are created and what is shared between them. You should have a basic understanding of a what a process is but we will not assume you’re an expert C programmer.

We will use the library procedure fork() so rst take a look at the manual pages. You will probably not understand everything they talk about but we get the important information that we need to start experimenting.

$ man fork

:

:

We see that fork requires the library unistd.h so we need to include this in our program. We also read that fork will return a value of type pid_t. This type is de ned in a header le included by unistd.h and is a way of making the code architecture independent. We will ignore this and assume that pid_t is a int. Further down the man pages we read that fork returns both the process identi er of the child process and zero. This is strange, how can a procedure return two di erent values? Let’s give it a try, create a le called dolly.c and write the following:

#include &lt;stdio .h&gt;

#include &lt;unistd .h&gt; int main () { int pid = fork ( ) ; printf (“pid = %d
” , pid );

return      0;

}

The above program will call fork and then print the returned value. Compile and run this program, what is happening?

<h1>                            2          The mother and the child</h1>

So the call to fork() somehow creates a duplicate of the executing process and the execution then continues in both copies. By looking at the returned value we can determine if we’re executing in the mother process or if we are in the child process. Try this extension to the program.

#include &lt;stdio .h&gt;

#include &lt;unistd .h&gt; int main () { int pid = fork ( ) ;

if ( pid == 0) {

printf (“I ’m the child %d
” , getpid ( ) ) ; } else { printf (“My child is called %d
” , pid );

}

printf (“That ’ s            it %d
” ,         getpid ( ) ) ;

<table width="560">

 <tbody>

  <tr>

   <td width="82">return }</td>

   <td width="478">0;This is not the only way this could have been implemented. One could for example have chosen to have a construction where we would provide a function that the child process would call. Di erent operating systems have chosen di erent strategies and Windows for example have chosen to provide a procedure that creates a new process that is independent of the mother process.2.1        wait a minuteTo terminate the program in a more controlled way we can have the mother wait() for the child process to terminate. Try the following:</td>

  </tr>

  <tr>

   <td colspan="2" width="560">#include &lt;stdio .h&gt;#include &lt;unistd .h&gt;#include &lt;sys/types .h&gt;#include &lt;sys/wait .h&gt;</td>

  </tr>

 </tbody>

</table>

int main () { int pid = fork ( ) ;

if ( pid == 0) {

printf (“I ’m the              child %d
” ,          getpid ( ) ) ;

sleep (1);

} else { printf (“My child is called %d
” , pid );

wait (NULL);

printf (“My child           has          terminated 
” );

}

printf (“This         is          the end (%d)
” ,            getpid ( ) ) ;

return      0;

}

The mother waits for its child process to terminate (actually it waits for any child it has spawned). Only then will it proceed, print out the last row and terminate.

<h2>                            2.2        returning a value</h2>

A process can produce a value (an integer) when it terminates and this value can be picked up by the mother process. If we change the program so that the child process returns 42 as it exits, the value can be picked up using the wait() procedure.

if ( pid == 0) {

return       42;

}     else     {

int res ; wait(&amp;res );

printf (“the          result             was %d
” , WEXITSTATUS( res ) ) ;

}

return      0;

<h2>                            2.3      a zombie</h2>

A zombie is a process that has terminated but whose parent process has not yet been informed. As long as the parent has not issued a call to wait() we need to keep part of the child process. When calling wait, the parent process should be able to pick up the exit status of the child process and possibly a return value. If the child process is completely removed from the system this information is lost.

We can see this in action if we terminate the child process but wait for a while before calling wait(). Do the following changes to the program, call it zombie.c, compile and run it in the background.

if ( pid == 0) { printf (“check the status 
” ); sleep (10);




printf (“and again
” );

return       42;

}     else      {

sleep (20); int         res ; wait(&amp;res );

printf (“the result was %d
” , WEXITSTATUS( res ) ) ; printf (“and again
” );

sleep (10);

}

return      0;

}

Check the status of the processes using the ps command. Notice how the two processes are created, how the child becomes a zombie and is then removed from the system once we have received the return value.

$ gcc -o zombie zombie.c

$ ./zombie&amp; :

$ ps -ao pid,stat,command :

<h2>                           2.4         a clone of the process</h2>

So we have created a child process that is a clone of the mother process. The child is a copy of the mother with an identical memory. We can exemplify this by showing that the child has access to the same data structures but that the structures are obviously just copies of the original data structures. Extend dolly.c and try the following:

int main ()            {

int pid ; int x = 123; pid = fork ( ) ;

<table width="309">

 <tbody>

  <tr>

   <td colspan="2" width="173">if ( pid == 0) {</td>

   <td width="19"></td>

   <td width="116"></td>

  </tr>

  <tr>

   <td width="104">printf (” x = 42;sleep (1);</td>

   <td width="69">child :</td>

   <td width="19">x</td>

   <td width="116">is %d
” , x );</td>

  </tr>

  <tr>

   <td width="104">printf (”}      else     {</td>

   <td width="69">child :</td>

   <td width="19">x</td>

   <td width="116">is %d
” , x );</td>

  </tr>

  <tr>

   <td width="104">printf (”x = 13;</td>

   <td width="69">mother :</td>

   <td width="19">x</td>

   <td width="116">is %d
” ,             x );</td>

  </tr>

 </tbody>

</table>

sleep (1);

printf (”          mother :     x       is %d
” ,          x );

wait (NULL);

}

return      0;

}

As you see both the mother and the child sees ha variable x as 123 but the changes made are only visible by themselves. If you want to see something very strange you can change the printout to also print the memory address of the variable x. Do this for both the mother and the child and you will see that they are actually referring to the same memory locations.

printf (”            child :         x         is %d and the           address       is         0x%p
” , x , &amp;x );

The explanation is that processes use virtual addresses and they are identical, they are however mapped to di erent real memory addresses. How this is achieved is nothing that we should explore now but its fun to see that it is working.

<h2>                           2.5        what we do share</h2>

Since the child process is a clone of the mother process we do actually share some parts. On thing that we do share are references to open les. When a process opens a le a le table entry is created by the operating system. The process is given a reference to this entry and this is reference is stored in a le descriptor table that is owned by the process. Now when the process is cloned, this table is copied and all the references are of course pointing to the same entries in the le table.

The standard output is of course nothing more than a entry in the le descriptor table so this is why both processes can write to the standard output. We also read from the same standard input so we have a race condition also there.

If you look at man pages you will see a whole range of structures that the processes share or not share but most of those are not very interesting to us in this set of experiments.

<h1>                          3            Groups, orphans, sessions and daemons</h1>

The mother, or should we call it parent process to be gender neutral, has a special relationship to the child process. The parent process has to keep track of its child’s and a child always knows the process identi er of its parent.

int main () { int pid = fork ( ) ;

if ( pid == 0) {

printf (“I ’m the               child %d with            parent %d
” ,           getpid () ,         getppid ( ) ) ;

} else { printf (“I ’m the parent %d with parent %d
” , getpid () , getppid ( ) ) ;

wait (NULL);

}

return      0;

}

Compile and run this in a terminal, who is the parent of the parent process? The following commands might give you a clue.

$ ps a

:

: $ echo $$

:

We could nd more information about the processes using some ags to ps. Try ps -fp $$ to see more information about the shell your using ($$ will expand to the process identi er of the shell). The PPID eld is the parent process identi er. Who is the parent of the shell? Where does it all stop?

<h2>                           3.1       the group</h2>

The fate of a parent and its child are not directly linked to each other but they belong to the same process group. Each process group has a process leader and in our simple examples the parent process has been the leader of the group. The group identi er/leader is retrieved using the system call getpgid().

int main () { int pid = fork ( ) ;

if ( pid == 0) {

int            child = getpid ( ) ;

printf (“I ’m the              child %d in            group %d
” ,         child ,          getpgid ( child ) ) ;

} else { int parent = getpid ( ) ;

printf (“I ’m the              parent %d in           group %d
” ,        parent ,              getpgid ( parent ) ) ;

wait (NULL);

}

return      0;

}




A group is treated as a unit by the shell, it can set a whole group to suspend, resume or run in the background (allowing the shell to use the standard input for interaction). We will however not go into how the shell is working so let’s just accept that processes belong to a process group.

<h2>                            3.2      orphans</h2>

As a change we can try to crash the parent process and see what happens with the child process.

#include &lt;stdio .h&gt;

#include &lt;unistd .h&gt;

#include &lt;sys/types .h&gt; #include &lt;sys/wait .h&gt; int main () { int pid = fork ( ) ;

<table width="621">

 <tbody>

  <tr>

   <td colspan="2" width="261">if ( pid == 0) {</td>

   <td rowspan="2" width="113"></td>

   <td rowspan="2" width="96"></td>

   <td rowspan="2" width="152"></td>

  </tr>

  <tr>

   <td width="35"></td>

   <td width="225">int             child = getpid ( ) ;</td>

  </tr>

  <tr>

   <td width="35"></td>

   <td width="225">printf (” child :   parent %d , sleep (4);</td>

   <td width="113">group %d
” ,</td>

   <td width="96">getppid () ,</td>

   <td width="152">getpgid ( child ) ) ;</td>

  </tr>

  <tr>

   <td width="35"></td>

   <td width="225">printf (” child :   parent %d , sleep (4);</td>

   <td width="113">group %d
” ,</td>

   <td width="96">getppid () ,</td>

   <td width="152">getpgid ( child ) ) ;</td>

  </tr>

  <tr>

   <td width="35"></td>

   <td width="225">printf (” child :                       parent %d ,</td>

   <td width="113">group %d
” ,</td>

   <td width="96">getppid () ,</td>

   <td width="152">getpgid ( child ) ) ;</td>

  </tr>

  <tr>

   <td width="35">}</td>

   <td width="225">else       {int             parent = getpid ( ) ;</td>

   <td width="113"></td>

   <td width="96"></td>

   <td width="152"></td>

  </tr>

  <tr>

   <td width="35">}</td>

   <td width="225">printf (“parent :                     parent %d ,sleep (2);int zero = 0; int i = 3 / zero ;</td>

   <td rowspan="2" width="113">group %d
” ,</td>

   <td rowspan="2" width="96">getppid () ,</td>

   <td rowspan="2" width="152">getpgid ( parent ) ) ;</td>

  </tr>

  <tr>

   <td colspan="2" width="261">        return      0;}</td>

  </tr>

  <tr>

   <td colspan="2" width="261">Save the program in a</td>

   <td colspan="3" width="361">le called orphan.c. Compile and execute the</td>

  </tr>

 </tbody>

</table>

program, notice how the parent identi er of the child process changes. The process has turned into an orphan and adopted by the upstart process (or init or systemd depending on which system you’re using). Note the new process identi er and then check its state using the ps command:

$ps &lt;whatever the process id was&gt; :

:

To see something fun you can take a look at the process tree of the process:

$pstree &lt;whatever the process id was&gt; :

:

<h2>                            3.3        sessions and daemons</h2>

The origin of the notion of a session is a user attaching and logging in to the system. A session consists of a set of groups and a session leader. As with groups, the sessions have identi ers that are equal to the leaders process identi er. Compile and run the program below, which process is the session leader of our processes?

#include &lt;stdio .h&gt;

#include &lt;unistd .h&gt;

#include &lt;sys/types .h&gt; #include &lt;sys/wait .h&gt; int main () { int pid = fork ( ) ;

if ( pid == 0) {

int            child = getpid ( ) ;

printf (” child :                 session %d
” ,             getsid ( child ) ) ;

} else { int parent = getpid ( ) ;

printf (“parent :              session %d
” ,            getsid ( parent ) ) ;

}

return      0;

}

When you start a new terminal, a new session is created. The operating system keeps track of sessions and will terminate all groups in a session if the session leader terminates. This means that if you log in to a system and start to run processes in the background they still belong to the same session as your login shell and will be terminated if the session terminates.

If one wants to create a process that should survive the session it must form its own session. It becomes a daemon, a process that is running in the background detached from any controlling terminal.

Many of the tasks performed by the operating system are performed by daemons. They keep track of network interfaces, USB devices or schedules tasks that should run periodically etc. Your system will probably have fty deamons running in the background but the consume very little resources.

<h1>                            4         Starting a program</h1>

So far we have seen how a process can be created and how the child process is related to its parent process. To understand how an operating system works there is one more very important functionality that we will take a look at how we create a process that will execute another program.

When you use the command shell this happens (almost) every time you enter a command. Some commands are interpreted by the shell and the shell will do something for us but most commands are actually programs that the shell will start for us. How is a program actually started?

<h2>                            4.1         transforming a process</h2>

In Unix systems the execution of a program is done by transforming an existing process to run the code of the given program. As you will see, starting a program is done in two steps – creating the new process and then transforming the process into executing the program.

The mechanism that makes this possible is the family of exec() system calls. Look-up the man pages of exec, we will use the one called execlp().

#include &lt;stdio .h&gt;

#include &lt;unistd .h&gt;

#include &lt;sys/types .h&gt; #include &lt;sys/wait .h&gt; int main () { int pid = fork ( ) ;

if ( pid == 0) {

execlp (” ls ” , ” ls ” , NULL);

printf (” this           will       only      happen     i f     exec       f a i l s 
” );

}     else     {

wait (NULL);

printf (“we ’ re         done
” );

}

return      0;

}

The call to execlp() will nd the program ls and then replace the code and data segments of the process with the code and data found in the executable binary. The stack and heap areas are reset so the program starts the execution from scratch.

<h2>                            4.2       redirection</h2>

Even if the memory segments of the process is cleared, the process keeps the le descriptor table. By changing the table entries we can make the program read from a standard input of our choice and we can of course redirect the standard output. This allows us to control the I/O operations of the program without changing the program in any way.

To see how this works we can create a small program that does nothing but writes to standard output. Let’s call this program boba.c.

#include &lt;stdio .h&gt;

int main () { printf (“Don ’ t get in my way.
” );

<table width="560">

 <tbody>

  <tr>

   <td width="82">return }</td>

   <td width="478">0;Now if we compile and run this program we will of course see the quote printed in the terminal.$gcc -o boba boba.c :$./boba :Note that we have to write ./boba and not simply boba if you have not set up your PATH variable to also include the current directory; more on this later.Now if we want to redirect the output to a le called quotes.txt we could of course do this from the shell directly.$./boba &gt; quotes.txt :To understand how the shell achieves this we could try to write a program jango.c, that clones itself, redirects the standard output and then transforms the clone into boba. Let’s go:</td>

  </tr>

  <tr>

   <td colspan="2" width="560">#include &lt;stdio .h&gt;#include &lt;unistd .h&gt;#include &lt;sys/types .h&gt;#include &lt;sys/ stat .h&gt; #include &lt;fcntl .h&gt;#include &lt;sys/wait .h&gt;</td>

  </tr>

 </tbody>

</table>

int main ()             {

int pid = fork ( ) ; if ( pid == 0) {

<table width="578">

 <tbody>

  <tr>

   <td colspan="2" width="578">       int                                           fd = open(“quotes . txt” , O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);</td>

  </tr>

  <tr>

   <td width="271">       dup2( fd ,       1);close ( fd );execl (“boba” , “boba” , NULL);</td>

   <td width="307"></td>

  </tr>

  <tr>

   <td width="271">           printf (” this          only             happens on}      else     {wait (NULL); }</td>

   <td width="307">f a i l u r e 
” );</td>

  </tr>

 </tbody>

</table>

return      0;

}

In the jango.c program we open a le quotes.txt (providing ags to open it in read-write mode and create it if does not exist). The operating system will grate a new le table entry and add a reference to it in our le descriptor table. The table entry will be the rst free entry in the table (3). We then use the system call dup2() to copy the entry to position 1 (the location of stdout). We then close the fd entry since we will not use this entry any more.

When we now call execl(), the process will turn into boba. The boba program knows nothing about what has happened but will of course direct all output to    le descriptor 1 as usual. Try it and you will see that the     le is created and that we will receive the output as expected.

<h2>                            4.3      pipes</h2>

The full beauty of how standard input and output can be redirected is shown when we introduce the concept of pipes. A pipe is a FIFO bu er of characters and when created we will allocated two le descriptor entries. One entry is for reading and the other for writing.

Since we are in full control over the descriptor table before we start executing a program, we can make one program send all the output to another programs input. From the shell this is a very powerful tool to combine sequences of commands.

Assume we have the commands (or rather programs) ps axo sid that will print the session identi er of every process in the system, sort -u that will sort lines and output only unique and wc -l that will count the number of lines. How do we combine these to nd the number of sessions in the system. Using the shell this is done in one line:

$ ps xao sid | sort -u | wc -l

:

This is achieved using pipes and we can set it up ourselves in a program. There are however a lot of details to get it right and we will explore this later in the course. For now you should explore using the pipes from the command line.

<h1>                            5       Summary</h1>

Processes are created by cloning an existing process, the execution continues in the two duplicates and the only way of telling in which copy we are executing in, is to look at the value returned from fork().

A parent and child process are in the same process group. If the group leader terminates all processes in the group will be sent a signal that will likely cause them to terminate.

Several groups belong to a session with a controlling terminal. If the session leader terminates or the controlling terminal closes, the whole session will be terminated.

A session that has been detached from any controlling terminal is called a daemon. Daemons handle many of the tasks that constitute an operating system.

Two copies have identical copies of le descriptor tables referring to the same le table entries. By changing the descriptor tables, the input and output of a process can be redirected. Two processes can use this to set up a pipe between them that acts as a bu ered FIFO channel.

A process can be transformed to run another program using the system call exec(). This will reset all memory segments but the transformed process keeps the le descriptor table.