## RMaxima Project Documentation ##

> |   **Authors**: Kseniia Shumelchyk and Hans W. Borchers <br>
<blockquote>|   <b>Date</b>:    August 20, 2012</blockquote>

<h3>Introduction</h3>

The goal of the project RMaxima is to create a package that allows to use from within R the capabilities of Maxima, a Computer Algebra System that is more powerful than the symbolic algebra systems available for R right now. The package should add new functionality to R for using symbolic expressions, such as equation solving, differentiation, integration, three-dimensional plotting, or multiple-precision arithmetics.<br>
<br>
<h3>Project Description</h3>

At the beginning of the project, existing approaches to communicate with the Maxima process were explored and analyzed  Two ways of interaction were identified: The first one based on TCP sockets, and the second one based on using the mechanism of pipes. Similar API BSD sockets are available in most modern operating systems, but some separate methods are required to start the Maxima process.<br>
<br>
In case of using unnamed pipes portability is provided automatically, but will depend on additional libraries. Based on some prior know-how it was natural to look at Boost.Process which is associated with, but not officially part of, the well-known C++ <a href='http://www.boost.org'>Boost</a> library. Possible alternatives were, among others, <a href='http://libexecstream.sourceforge.net/'>Libexecstream</a> or <a href='http://pstreams.sourceforge.net/'>PStreams</a>.<br>
<br>
For the development of this package unnamed pipes were chosen as a way of allowing R to interact with the Maxima process. After connecting to Maxima there is the challenge of processing the output of Maxima. The communication protocol via Maxima console is in principle quite simple: The user repeatedly enters expressions and Maxima produces some result and returns it as a string. This message exchange pattern may be described as 'request-response'.<br>
<br>
To simplify the processing, Maxima output is controlled by an option file (<code>display.lisp</code>). It configures Maxima output in such way that allows to identify different parts of the output coming from Maxima. Result expressions are marked in a special way and displayed in the one-dimensional, i.e. non-graphical mode. Parsing the output is realized through regular expressions provided by the library Boost.Regex.<br>
<br>
All this has been implemented in a demo system at the end of the first coding phase, except that the communication between R and Maxima consoles is more batch-like and not yet interactive. During the second coding phase, a true request-response interaction between the R console and the Maxima process has been realized with the help of the <code>Rcpp</code> package.<br>
<br>
The original project plan, taken from the project proposal, is enclosed here.<br>
<br>
<b>Project Plan (first phase)</b>

<table><thead><th> <b>Week</b> </th><th> <b>Timeline</b> </th><th> <b>Activity</b> </th></thead><tbody>
<tr><td> 1 </td><td> April 23 - May 10   </td><td> Reading the documentation of Maxima and R, getting familiar with the project.  </td></tr>
<tr><td> 2 </td><td> May 11 - May 20     </td><td> Discussing a project architecture with a mentor.  </td></tr>
<tr><td> 3 </td><td> May 21 - June 3     </td><td> Creating a simple Maxima "communication manager" with unit tests.  </td></tr>
<tr><td> 4 </td><td> June 4 - June 10    </td><td> Code review of the managers code; testing and bugfixes  </td></tr>
<tr><td> 5 </td><td> June 11 - June 19   </td><td> Extending Maxima communication manager on Windows and Mac OS  </td></tr>
<tr><td> 6 </td><td> June 20 - June 22   </td><td> Testing and bugfix  </td></tr>
<tr><td> 7 </td><td> June 23 - July 2    </td><td> Adding ability to send commands from R project interpreter to maxima and getting the result  </td></tr>
<tr><td> 8 </td><td> July 3 - July 4     </td><td> Code review and making changes  </td></tr>
<tr><td> 9 </td><td> July 5 - July 8     </td><td> Testing and bugfix  </td></tr></tbody></table>

<b>Project Plan (second phase)</b>

<table><thead><th> <b>Week</b> </th><th> <b>Timeline</b> </th><th> <b>Activity</b> </th></thead><tbody>
<tr><td> 10 </td><td> July 16 - July 20 </td><td> Controlling Maxima (start, stop) and user interactivity </td></tr>
<tr><td> 11 </td><td> Juli 23 - July 27 </td><td> Handling Maxima error messages and Maxima plots </td></tr>
<tr><td> 12 </td><td> July 30 - August 3 </td><td> Returning Maxima results to R; filtering commands </td></tr>
<tr><td> 13 </td><td> August 6 - August 10 </td><td> Using Rcpp for keeping C++ pointers </td></tr>
<tr><td> 14 </td><td> August 13 - August 17 </td><td> RMaxima Vignette; code documentation </td></tr></tbody></table>


<h3>Project Results (1st Coding Phase)</h3>

The result of the first coding phase of the project is a demo program proving the feasibility of the approach described above. When all tools needed are installed (Boost, Boost process library, Maxima) as listed below, the demo library can be compiled as a shared library for R and then dynamically loaded into R with the <code>dyn.load()</code> function.<br>
<br>
The demo program will automatically load the RMaxima shared library and then send some simple commands to Maxima. The result strings returned from Maxima will be displayed on the R command terminal. Here is an example, with the symbolic commands <code>float</code>, <code>sin + cos</code>, <code>plot2d</code>, and <code>diff</code> send to Maxima:<br>
<br>
<pre><code><br>
&gt; source("maximaexe.R")<br>
# Calling C function<br>
&gt;&gt;&gt; float(1/3);<br>
0.33333333333333<br>
# Returning to R<br>
<br>
# Calling C function<br>
&gt;&gt;&gt; sin(%pi/2) + cos(%pi/3);<br>
3/2<br>
# Returning to R<br>
<br>
# Calling C function<br>
&gt;&gt;&gt; plot2d(x^2 - x + 3, [x, -10, 10]);<br>
""<br>
# Returning to R<br>
<br>
# Calling C function<br>
&gt;&gt;&gt; diff(sin(x), x);<br>
cos(x)<br>
# Returning to R<br>
</code></pre>

The graphics window, opened by Maxima through the <code>plot2d</code> command, will only shortly pop up and then disappear again. It has to be seen how graphical output from Maxima can be kept open until the user closes the window..<br>
<br>
The other commands return correct results from Maxima, at the moment available only as strings. To call Maxima interactively from the R command line will be one of the next tasks.<br>
<br>
<br>
<h3>Project Results (2nd Coding Phase)</h3>

The result of the second coding phase of the project is a program providing mechanism of interaction with CAS Maxima. When all tools needed are installed (Boost, Boost process library, Maxima, Rcpp package) as listed below, the library can be compiled as a shared library for R and then used from R as Rcpp module.<br>
<br>
The program requires some initial R command that allows further using Maxima in interactive mode.<br>
<br>
<pre><code><br>
require(Rcpp)<br>
m &lt;- Module("Maxima", dyn.load("Rmaxima.so"))<br>
mx.start &lt;- function() {<br>
mxm &lt;&lt;- new(m$RMaxima)<br>
}<br>
mx.exec &lt;- function(x) {<br>
mxm$execute(as.character(x))<br>
}<br>
</code></pre>

Before executing Maxima we need to start Maxima using command:<br>
<br>
<pre><code><br>
mx.start()</code></pre>

After Maxima is started we can send any commands to Maxima. The result strings returned from Maxima will be displayed on the R command terminal. Also we can keep results obtained as a strings in R variables.<br>
<br>
Below is an example, with the symbolic commands float, sin + cos,plot2d, diff send to Maxima. Also RMaxima allows to refer to the latest result through the % character, and to any previous input or output by its respective prompted %i (input) or %o (output). For example:<br>
<br>
<pre><code><br>
&gt; mx.exec("9+7")<br>
[1] "16"<br>
&gt; mx.exec("% - 10")<br>
[1] "6"<br>
&gt; mx.exec("%o1 * 2")<br>
[1] "32"<br>
&gt; mx.exec("float(1/3)")<br>
[1] "0.33333333333333"<br>
&gt; mx.exec("sin(%pi/2)+cos(%pi/3)")<br>
[1] "3/2"<br>
&gt; result = mx.exec("diff(sin(x),x)")<br>
&gt; result<br>
[1] "cos(x)"<br>
</code></pre>

The graphics window, opened by Maxima through the plot2d or plot3d commands, is seen how graphical output from Maxima and stay open until the user closes it manually. A new plot command will overwrite the old plot.<br>
Here is the examples of executing plot commands:<br>
<br>
<pre><code><br>
&gt; mx.exec("plot2d(x^2-x+3,[x,-10,10])")<br>
[1] "\"\""<br>
&gt; mx.exec("f(x,y):= sin(x)+cos(y)")<br>
[1] "f(x,y):=sin(x)+cos(y)"<br>
&gt; mx.exec("plot3d(f(x,y),[x,-5,5],[y,-5,5])")<br>
[1] "false"<br>
</code></pre>

Maxima calls Gnuplot for plotting, so you will see the following graphics windows. The 3d graph can be zoomed and rotated in three-dimensional space by the user.<br>
<br>
<img src='http://rmaxima.googlecode.com/files/plot2d.png' />

<img src='http://rmaxima.googlecode.com/files/plot3d.png' />

The other commands return correct results from Maxima, at the moment available only as strings.<br>
<br>
RMaxima handle error messages generated by Maxima as well as errors that are generated by when the calls to Maxima returns an error:<br>
<br>
<pre><code><br>
&gt; mx.exec("2+2;;")<br>
Error in mxm$execute(as.character(x)) :<br>
Bad expression: only one ;|$ terminated expression at a time is allowed<br>
&gt; mx.exec("asd+-*;")<br>
Error in mxm$execute(as.character(x)) :<br>
Maxima error: incorrect syntax: * is not a prefix operator<br>
asd+-*;<br>
</code></pre>

Created program allows to call Maxima commands interactively, one command after the other, defining variables that are still alive during the next user command.<br>
<br>
<br>
<h3>Problems Encountered</h3>

<h4>Maxima</h4>

Automatic processing of Maxima console output is quite complex: it's difficult to impose a general structure on Maxima output that will be valid for all kinds of requests sent to Maxima. According to documentation of the Maxima distribution the system has a number of variables that control the allocation of significant fragments of console output, which can significantly simplify the automatic processing of Maxima output. Here are the variables we used and their meanings:<br>
<br>
<blockquote>(1) <code>prompt-prefix</code> - the string is printed before each input request;<br>
(2) <code>prompt-suffix</code> - the string is printed after each input request;<br>
(3) <code>alt-display2d</code> - a function that replaces the standard Maxima in two-dimensional mode when printing expressions.</blockquote>

Setting these variables happens in the file <code>display.lisp</code>, written in Common Lisp (the programming language the kernel of Maxima is written in), and the path to that file is provided with the option <code>.p &lt;File Path&gt;</code>. The content of this file is loaded during Maxima booting.<br>
<br>
<h4>Boost</h4>

Also there were problems with using the Boost.Process library. For correctly working functions from Boost.Process required the Boost.Filesystem version 2, but latest Boost versions by default use version 3. So for using Boost.Process these things had to be fixed manually, and the fixes are included in the Boost Process files distributed with RMaxima.<br>
<br>
The <a href='http://www.highscore.de/boost/process/'>Boost Process</a> library was rejected by the Boost core team in March 2011 and has since then not been included into Boost officially. A consequence is that it must be downloaded and installed separately. That poses the problem of how to make RMaxima become a package distributed on CRAN. By the way, we were in contact with the developer of Boost.Process and he indicated he is still working on the library at times.<br>
<br>
[Remark: As Dirk Edelbuettel told us, the CRAN platform for Linux is "Debian testing" which includes 'boostlib', the standard Boost library. For Windows he provides Uwe Ligges with files and Boost headers such that the compilation process works on that platform too. (Mac OS X ?)]<br>
<br>
<h4>Interactive Mode</h4>

There were several different ideas about implementing interactive mode. The main question was how to correctly store C++ object in R.<br>
<br>
The first version of interactive mode (RMaxima-0.2) based on creating mx.exec() function that call C++ function, execute Maxima command and get result. But it was not really interactive because Maxima process was alive only for each C++ command called from R.<br>
<br>
The second approach was based on supporting Asynchronous input/output operations for Maxima process. But after research this approach was rejected. At first, Boost.Process library don’t have appropriate mechanism and uses Boost.Asio library for Asynchronous I/O. At second, this approach is also not guarantee interactive mode.<br>
<br>
After some additional survey we decide to go forward with Rcpp package that allows store C++ objects in R and expose C++ classes in R through Rcpp modules (RMaxima-0.3). For our further goals using Rcpp would be a very helpful, for example in using and keeping Maxima results in R.<br>
<br>
<br>
<h3>Next Steps</h3>

The RMaxima GSoC project has reached its aim to construct an interprocess communication between R and another process running Maxima. Command strings can be sent from R to Maxima, and returned results will be displayd in the R command terminal.<br>
<br>
The following goals appear as natural follow-ups for a later continuation of the project:<br>
<br>
<ul><li>Filtering commands, e.g. <code>mx.integrate()</code> for symbolic integration, or <code>mx.plot()</code> for plots; how will a set of commands be combined; what about errors and error mesages generated by Maxima, shall they be proprocessed.</li></ul>

<ul><li>Extend to Windows, i.e. what is needed to generate a binary Windows version, even if this cannot be provided as a binary package through CRAN. Some steps in the direction of making it a package, for instance, how can the Boost process library be distributed with the package.</li></ul>

<ul><li>Use Maxima results in R, i.e. use floating point numbers returned from Maxima in R calculations, or convert function expressions returned from Maxima into R functions. How can high-accuracy numbers returned from Maxima be utilized in R through the Rmpfr package.</li></ul>


<h3>Installation</h3>

This installation procedure for RMaxima works only on Linux systems and has been tested on Ubuntu, Mint, and Debian Linux ("squeeze"). It is assumed that a newer version of R (>= 2.15.1) is installed as usual, and that the normal C++ development environment with 'gcc' etc. is available on the Linux system. The installed Rcpp package should be >= 0.9.13<br>
<br>
<ul><li>Install the Boost libraries (>= 1.42), the preferred way is to use the Synaptic package manager, you can also try<br><code>sudo apt-get install libboost-dev</code><br><br>Make sure that the Boost libraries <code>boost_filesystem</code> and <code>boost_regex</code> are included with the installation.</li></ul>

<ul><li>Install the Boost process library (but not the new beta version 0.5). The file provided is <code>BoostProcess.tar.gz</code> that needs to be expanded. The subdirectory <code>boost/process</code> shall be copied to the Boost directory on the Linux system, i.e. <code>/usr/include/boost/process/</code>, the file <code>process.hpp</code> into <code>/usr/include/boost/</code> <br><br>As administrator rights required, for this action you could apply the <code>sudo nautilus</code> command.<br>Don't touch the <code>libs</code> directory, it is meant for installing on Windows.</li></ul>

<ul><li>Install Maxima (>= 5.21) for the Linux system. Usually this is best be done through the Synaptic package manager, or  with <code>sudo apt-get install maxima</code>. Exporting the <code>MAXIMA_PATH</code> is no longer necessary.</li></ul>

We are now ready to install and run the RMaxima demo program:<br>
<br>
<ul><li>Unpack the files included in the file <code>RMaxima-0.3.1</code> into a directory, would be best to call that <code>RMaxima</code> too. The directory will contain the following files:<br>
<pre><code>        display.lisp    MaximaChain.cpp     MaximaChain.h<br>
        Rmaxima.cpp     Rmaxima.R<br>
</code></pre></li></ul>

<ul><li>Go to this directory (using a command terminal) and compile the RMaxima library with the following command:<br>
<pre><code>    RCPP_CXXFLAGS=`Rscript -e 'Rcpp:::CxxFlags()'` \<br>
    RCPP_LIBS=`Rscript -e 'Rcpp:::LdFlags()'` \<br>
    PKG_CPPFLAGS="${RCPP_CXXFLAGS}" \<br>
    PKG_LIBS="-lboost_filesystem -lboost_system -lboost_regex ${RCPP_LIBS}" \<br>
    R CMD SHLIB imaxima.cpp MaximaChain.cpp<br>
</code></pre>
If no error occurs, this command will, among others, generate a shared library called <code>maximaexe.so</code>.</li></ul>

<ul><li>Start R (with the <code>R</code> command) in this directory and source the R script file that calls Maxima with some fixed demo commands  and returns Maxima's results:<br>
<pre><code>        source("Rmaxima.R")<br>
</code></pre>
This will load the shared library and display the interaction with Maxima through fixed commands, as described in section "Project Results" above.</li></ul>

If starting RMaxima by hand, use the following commands (or put them in a starting procedure of your own):<br>
<br>
<pre><code><br>
&gt; library("Rcpp")<br>
&gt; m &lt;- Module("Maxima", dyn.load("Rmaxima.so"))<br>
&gt; mx.start &lt;- function() mxm &lt;&lt;- new(m$RMaxima)<br>
&gt; mx.exec &lt;- function(s) mxm$execute(as.character(s))<br>
&gt; mx.start()<br>
&gt; mx.exec("...")  # valid Maxima command<br>
</code></pre>

After Maxima started, you can try to execute Maxima commands through the <code>mx.exec("...")</code> where "..." shall be a syntactically valid Maxima command. See the examples in the 'Project Results' section above.<br>
<br>
<br>
<h3>Project Details</h3>

Interaction with the Maxima has been implemented as a single class called <code>MaximaChain</code> that allows to separate the functionality that is responsible for interaction with the Maxima process and the code that provides integration with R. For creating a new connection to the Maxima that’s enough to create an instance of the <code>MaximaChain</code> class.<br>
<br>
<code>MaximaChain</code> class represents a proxy to an instance of a Maxima process.<code>MaximaChain</code> allows interaction with the Maxima at more higher level than approach with the sockets. User just can send a command and receive a result rather than write crude bytes to a socket or a pipe connected to the Maxima process's input/output streams. To kill a Maxima process you should delete <code>MaximaChain</code> object.<br>
<br>
The list of general tasks is:<br>
<br>
<ol><li>Launching Maxima process and keeping it for sequential user input;<br>
</li><li>Ability to send command to Maxima and return result in convenient format;<br>
</li><li>Handling errors messages generated by Maxima and errors that occurs after command execution (when call to Maxima returns an error);<br>
</li><li>Checking the validity of input expressions;<br>
</li><li>Correctly stopping Maxima process and removing appropriate object without memory and resource leaks.</li></ol>

The main functions and methods implemented in <code>MaximaChain</code> class that provides listed requirements are<br>
<br>
1. <code>MaximaChain</code> constructor.<br>
Creates an object to interact with Maxima and launches Maxima as a child process.<br>
Have three parameters: full path to Maxima executable file, path to directory for output files, path to directory that contains supporting files.<br>
<br>
2. <code>MaximaChain</code> destructor.<br>
Only sending to Maxima process command “quit()” not enough to destroy MaximaChain object because Windows cleans up resources only when all handles are closed. MaximaChain destructor closes all handles and waits for child process termination.<br>
<br>
3. <code>executeCommand</code> function.<br>
Sends command to Maxima and returns Maxima output after last command and ending just before the prompt starts.<br>
<br>
4. <code>executeCommandList</code> function.<br>
Passes its argument as the following command of Maxima process and returns the entire response to this command Maxima, including errors.<br>
<br>
5. <code>sendCommand</code> function.<br>
Just send a command to the Maxima. Command must end with ';' or '$'. In another case we append ';' character.<br>
<br>
6. <code>checkInput function</code>.<br>
Check if the input expression is valid. Initialized by a zero value.<br>
Have two parameters: next expression character to process and current checker state.<br>
Return new checker state. The final state is STATE_END.<br>
<br>
7. <code>Struct</code> Reply.<br>
Reads Maxima output until prompt is found and then parses it. Have one input parameter - input stream where Maxima writes its output.<br>
<br>
For manipulating C++ objects in R were used Rcpp modules. This approach allows create instances of C++ classes, retrieve/set data members and call methods. External pointers are also useful for that but Rcpp modules wraps them in a nice to use abstraction.<br>
<br>
The module is created in a C++ file using the RCPP_MODULE macro, which then contains declarative code of what the module exposes to R.<br>
<br>
Since C++ does not have reflection capabilities, modules need declarations of what to expose: constructors, fields or properties, methods or finalizers.<br>
<br>
See "Exposing C++ functions and classes with Rcpp modules", Dirk Eddelbuettel and Romain Francois, July 9, 2010.<br>
<br>
For using MaximaChain instance and handling errors was created suitable interface in RMaxima C++ class and Rcpp module called Maxima for exposing this functionality to R.<br>
<br>
<pre><code><br>
RCPP_MODULE(Maxima)<br>
{<br>
class_&lt;RMaxima&gt;( "RMaxima")<br>
.constructor()<br>
.method("execute", &amp;RMaxima::execute)<br>
.finalizer(&amp;rmaxima_finalizer) ;<br>
}<br>
</code></pre>

When the R reference object that wraps the internal C++ object goes out of scope, it becomes candidate for GC. When it is GC’ed, the destructor of target class is called. Finalizers allows to add behavior right before the destructor is called.<br>
<br>
For using Maxima process in R were implemented following functions:<br>
<br>
<ul><li>mx.start() that creates an instance of the Maxima module and launches the Maxima process;<br>
</li><li>mx.exec(character(x)) that executes a Maxima command and returns the result to R.</li></ul>

Stopping Maxima manually and integrating code into an R package will be done in one of the next versions.