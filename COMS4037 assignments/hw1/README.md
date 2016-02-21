# Homework 1: Web log data wrangling

### This is an individual assignment
### Points: 10% of your final grade
### Due: Monday, 29 February, at 11:59 pm SAST

**This assignment has been originally developed for [UC Berkeley CS186
   course](http://www.cs186berkeley.net/); we use it for COMS4037 with
   their gracious permission**

## Introduction

This assignment is meant to serve two purposes:

1. To acquaint you with a typical task in data management: using
command-line tools in combination with a scripting language to
"wrangle" publicly-available data into a structured format suitable
for subsequent analysis.

2. To acquaint you with working with datasets that are too large to
   fit into main memory.

## Your task

Given a file containing web access logs for a YouTube video, you have
to generate [csv](http://en.wikipedia.org/wiki/Comma-separated_values)
files that capture information about user sessions in a structured
format, so that it can be later analised using a DBMS or a statistical
package.

For this assignment, you are limited to using
[Python 2](https://www.python.org/), the
[bash shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell)), and
[standard Unix utilities](http://en.wikipedia.org/wiki/List_of_Unix_utilities).
You have to write your  code in a
[Jupyter notebook](http://jupyter.org/), which makes it easy to
interactively manipulate code and data, providing you with immediate
feedback on whether your code is doing what you want it to do.

Your script should be written so that it can handle an input file that
is larger than the main memory of the computer that runs the script,
making no assumptions about the size of the main memory.  Thus, you
should

1. write code that requires only a fraction
of the data to be in memory at any given time;

2. use UNIX utilities, such as `sort`, that implement out-of-core
divide-and-conquer algorithms.

You don't have to write very complicated code either in Python or in
bash: take advantage of UNIX utilities as much as possible.  In
particular, to complete this homework, you don't need to
implement any out-of-core algorithms: UNIX utilities can do all the
out-of-core processing for you.

## Getting started

Pull the latest changes from the `hw` repository. Having done that,
you should have `hw1` directory in your repository, with the following
files in it:

1. `hw1.ipynb`

2. `apachetime.py`

3. `test_memory_usage.sh`

To access the dataset you will be working with, log into your account
on a lab machine, and go to the `~/coms4037/hw1` directory.
Alternatively, you can download it from
[here](http://www.cs.wits.ac.za/~dmitry/coms4037/files/hw1_DATA_DIR.zip).

*Note*: Whether you are working on a lab machine or your personal
computer, the dataset should reside in the `~/coms4037/hw1` directory;
if you want to change the location of the dataset, make sure that you
edit the code in hw1.ipynb accordingly.


## Running Jupyter Notebook

*Note*: in the following instructions, 'Notebook' -- used
 interchangeably with 'Jupyter Notebook' -- refers to an application,
 while 'a notebook' refers to a file you work on using the
 application.

Jupyter Notebook is an application with a web-browser-based interface
that runs on top of a kernel that performs the computations requested
through the interface.  There exist multiple kernels, implemented in a
variety of languages, thus allowing you to use the Notebook to execute
code written in any of those languages.  The kernel that executes
Python code is [IPython](http://ipython.org/) -- thus, in this
assignment, you will be using the Notebook to work with IPython.  To
start working with the Notebook, you need to `cd` into your `hw/hw1`
directory and run

    jupyter notebook

at the shell prompt. This should automatically open a new tab with the
 Notebook dashboard in your browser; as the Notebook interface opens
 in a browser, the server process will take over the terminal.  In
 case a new tab with a dashboard had not opened automatically, open a
 new browser tab and point it at http://localhost:8888, which is the
 port where the server is running.

To begin this assignment, click on the `hw1.ipynb` file in the "Files"
tab of the dashboard.  As you work with a notebook, remember to save
your work by pressing Ctrl-S.

Once you're done using the Notebook, you should down the server
running in your terminal by pressing Ctrl-C on the keyboard.

## Python 2 vs Python 3

We use Python 2 for this assignment.  The lab machines have the
anaconda distribution of Python, which includes Jupyter Notebook,
IPython, as well as all the libraries from the so-called PyData
stack. The machines have both anaconda and anaconda3 installed -- the
former runs Python 2, while the latter by default runs Python 3. If
you have anaconda3 installed on your personal computer, you can switch
to using Python 2 by enabling the py2 environment; to do so, type

    source activate py2

at the shell prompt.  To switch back to Python 3, type

    source deactivate

at the shell prompt.

##Specification

Your solution should be driven by the `process_logs( )` function in
hw1.ipynb, which expects one argument -- an iterator (or stream) of log
lines to be parsed. This input stream can only be iterated over
once.

The web logs you are going to be working with were produced by an
Apache web server.  Each line represents a request to the server that
hosted the video access to which we are analysing. Each log has the
following format:

    24.71.223.142 - - [28/Apr/2003:16:24:59 -0700] "GET /random/video/puppy.mov HTTP/1.1" 200 567581 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)"

This format is called "Combined Log Format";  you can find a
description of each of the fields
[here](https://httpd.apache.org/docs/1.3/logs.html#common).

What we are doing in this assignment is usually referred to as
[session reconstruction](https://en.wikipedia.org/wiki/Session_(web_analytics)#Session_reconstruction). A
web session represents a user's actions on a website in a consecutive
chunk of time.  Session information is useful for tracking the path a
user took through the site, as well as for collecting various metrics
related to the user behaviour.  We will use the server logs to
reconstruct user sessions.

We adopt a time-oriented approach to reconstructing the sessions; that
is, requests made by the same IP address within a specified time
period, which in our case is going to be 30 minutes, are considered
part of the same session.  Thus, if an IP address does not make a
request within 30 minutes (*inactivity threshold*) from the previous
request, we consider that IP address's next request to be the start of
a new session.

We are interested in computing session lengths, as well as the number
of requests made in each session.

Thus, `process_logs()` should produce 3 csv files:

* `hits.csv` has the header `ip,timestamp`, where
    * each row corresponds to a request made to the web server;
	* `ip` is an IP address that made the request;
    * `timestamp` is a
      [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time), which
      reflects the number of seconds elapsed between 12 am on 1
      January 1970 and the time the request was made;
    * each row corresponds to one row in the input log file, and the
      order of the rows matches the order of the log file entries.
	  
* `sessions.csv` has the header `ip,session_length,num_hits`, where
    * `ip` is an IP address associated with a session;
    * `session_length` is the length of a session measured in seconds
      (bear in mind that our inactivity threshold is 30 minutes;
      thus, a request made exactly 30 minutes after the previous one
      is considered to be part of the same session);
    * the order of the rows is immaterial.
	
* `session_length_plot.csv` is a file representing a histogram-style
  plot, with the sizes of the bins increasing exponentially; it has
  the header `left,right,count`, where
    * in each row, the first two columns describe a range [`left`, `right`);
	* `count` is the number of sessions whose length falls within that range;
    * the left and right boundaries of each bin are powers of 2 with the exception that
	   the left boundary of the first range should be 0
        * e.g. [0, 2), [2, 4), [4, 8), [8, 16), ...
    * rows with the value of `count` equal to 0 should be omitted;
    * the rows should be sorted in the ascending order according to
	   the value contained in the `left` field.

## Testing

The reference output is available on lab machines in the
directory `~/coms4037/hw1/ref_output_small`.  You can make use of these files in two ways.

First, you can take a look at them to make sure you understand the format of the csv files your code is supposed to produce.

Second, the `hw1.ipynb` file contains code that tests the output
produced by your code against the reference output -- you can run it
to make sure that your code is correct (for more details, read the
notes in the `hw1.ipynb` file).

You need to make sure that your code will scale to datasets that are
bigger than memory, regardless of how large or skewed the dataset is or how
much memory the machine running your code has.  To help you assess
your implementation, you can use a test script
`test_memory_usage.sh`, which uses the [`ulimit
-v`](http://ss64.com/bash/ulimit.html) command to cap the amount of
virtual memory your code can use.  If you get a memory error, then
your code is not doing appropriate streaming and/or
divide-and-conquer.

##Notes
* Python has a useful [CSV
library](https://docs.python.org/2/library/csv.html) for csv
manipulation -- it will make your life a lot easier.

* As in the sample csv files, the line endings for your output files
  should have UNIX line-endings (\\n); be careful, as this is not the
  default line ending for a Python's `csvwriter` object.

* The UNIX utilities are written in C -- therefore, they are much
  faster than anything you will write in Python; moreover, they are
  time-tested and thus free of bugs. This, if a sub-problem can be
  solved with a UNIX utility, we strongly recommend that you use it.
  Notes in `hw1.ipynb` tell you how to access Unix utilities from
  IPython.

* When using shell commands, rather than writing out intermediate
  results to temporary files, use
  [UNIX pipes](http://en.wikipedia.org/wiki/Pipeline_(Unix)) to chain
  the output of one command to the input of the other.

* The reference implementation finishes `process_logs_small()` in less
  than a minute, and `process_logs_large()` in less than 3
  minutes.  Your code will be tested with reasonable, but generous
  timeouts -- at least 2x our reference times.

* You will be graded both on correctness of output and efficient
  memory usage.

## Submission instructions

Follow the submission instructions in
[HW0](https://github.com/WITS-COMS4037/hw/blob/master/hw0/README.md).

## Good luck!


