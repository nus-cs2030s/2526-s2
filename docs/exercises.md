# Guide to Programming Exercises

Programming exercises aim to help you practice the concepts taught in the course.  They are designed to be completed within a day.  You are encouraged to discuss and seek help from others if you get stuck.  However, you should ensure that you understand the resulting code and the concepts behind them, and not just copying the code blindly.

## GitHub Setup 

You need a one-time setup at the beginning of the semester to link your PE account to your GitHub account.  Follow [the instructions here](github.md) to set up your GitHub account for CS2030S.

## Vim Setup 

You need a one-time setup at the beginning of the semester to install the standard vim configuration, color schemes, and plugins.  Follow [the instructions here](vim/setup.md) to set up your Vim for CS2030S.

You will not be able to retrieve an exercise if the expected Vim-related directory cannot be found. 

## GitHub Classroom

We will use GitHub Classroom for our exercise release and submission for CS2030S.

Here are what you need to do for every exercise:

### 1. Accept the Exercise

Make sure that you have logged into GitHub.

_If you have multiple GitHub accounts, make sure you use the one with the same GitHub username you have submitted to us_.

Click on the given URL to accept the exercise. 

A repo will be created automatically for you.

!!! warning "WARNING"
    Do not interact with the repo directory using GitHub or other `git` commands.

### 3. Get a Copy on PE Hosts

Run the command `/opt/course/cs2030s/get exX` (where X is the exercise number) to clone a copy of the exercise on your home directory.  You will see a new directory named something like `exX-username` created, with the skeleton files inside.

#### Read the Task

- The exercise task will be given in a markdown file named `exX-task.md` in the exercise directory.
- You can open it using the command `view exX-task.md` or `/opt/course/cs2030s/bin/glow -p exX-task.md`.  Alternatively, you can view it through the web browser on your GitHub repository.  We suggest that you get use to viewing it on the command line, as you will need to do so during the practical exams.
- Read through the question carefully before starting to complete the task.

#### Solving the Task

You need to edit, compile, run, and test your code on the PE hosts.

!!! warning "WARNING"
    Do not edit your code directly on GitHub.

#### Testing your Code

In CS2030S exercises, there are two types of test cases: unit tests to test individual classes or methods, and integration tests to test the whole program.  The task statements will indicate which type of tests are provided for each exercise.

- The unit tests are given as part of the skeleton code in a file named `exX-test.jar`.  The command to run the unit tests is given in the task statements.

- The integration tests are given as input and expected output files in the `tests` sub-directory.  The command to run the integration tests is given in the task statements.  Typically, you run `/opt/course/cs2030s/test-main exX` to run the integration tests.

Students typically go through the edit-compile-test cycle multiple times to complete the exercise.


### 4. Submit a Copy 

When you are ready to submit, run `/opt/courss/cs2030s/submit exX` (where X is the exercise number).  This will do the following:

- Reformat your code according to the style guide 
- Runs a checker on your code (see details below)
- Generate a report 
- Submit a copy of the code and the report to GitHub.  

You can submit multiple times, but your tutor is only obligated to read the last copy submitted before the deadline.

If your submission is successful, you will see an output like this:
```
submitting <exercise> for <your name>
reformatting Java files...done
compiling...done
testing individual classes...done
testing Main...done
linting...done
checking style...done
generating report...done
submitting...
updating master branch...
  :
  :
pushing to GitHub...
submitted.
```

If your code generates any warning or error during the pre-submission check, the errors will be displayed on the terminal.  You should fix them and submit again.

!!! warning "WARNING"
    Do not use `git push` or other `git` commands to submit your code to GitHub.

!!! tips "Alias"
    You may shorten the commands by creating aliases:

    ```shell
    alias get="/opt/course/cs2030s/get"
    alias submit="/opt/course/cs2030s/submit"
    ```

#### Pre-submission Check 

The pre-submission check runs the following to properly validate your code before submission:

1. It checks if your code compiles without errors.
2. It runs all unit tests and integration tests (where applicable). 
3. It checks if your code follows the style guide (we used `checkstyle` with a custom configuration)
4. It runs a linter to check for common mistakes (we used `pmd` with a custom configuration)

!!! warning "WARNING"
 
    In some exercises, some support files are provided to you as part of the skeleton code.  The task statements will clearly state that you are not supposed to modify these files.  If you run the `check` command, it may restore these files to their original versions.

If you want to run the checker on your own without submission, run `/opt/course/cs2030s/check exX` (where X is the exercise number).  

    After this, you can simply run `get exX`, `submit exX`, or `check exX`.  To setup these aliases automatically, you can put the three lines above in your `~/.bash_profile` (it will take effect the next time you log in).  

You should fix any compilation errors, test failures, style errors, or linter warnings before submission.

### 5. Receiving Feedback

The tutors will provide feedback on your submission via Github after the deadline.  You can reply to their comment, etc, on GitHub as well. 

!!! warning "WARNING"
    You should not change your code on GitHub after the deadline (by either re-running `submit` or using `git` commands directly) to avoid interfering with the feedback process.

### 6. Feedback Report and Achievement Badges

A file named `feedback.md` that contains auto-graded output will be placed into your GitHub repo after your submission.  

We will assign an achievement badge, which can be one of the following:

  * **Excellent** Compiles without warning or style errors. Pass all test cases.
  * **Good** Compiles with one or more warnings or style errors. Pass all test cases.
  * **Need Improvement** Fail one or more test cases (including internal test cases)
  * **N/A** Late submission; Submitted skeleton only; No submission; Submitted non-compilable code.

These achievement badges help us keep track of the students' progress and give students a sense of their learning.  They do not contribute to the final grade.

### Warning

Let us repeat: You should only interact with your submissions on GitHub using the provided scripts `get` and `submit`.  Failure to do so will break our workflow and will not be appreciated.

If you accidentally break your repo by running `git` commands on it or edit it directly on GitHub, you should save a copy of your code elsewhere, then reset your exercise directory, by (i) requesting your tutor to delete the repo on GitHub, (ii) deleting the corrupted directory on PE nodes, (iii) go through Steps 1 and 2 again, then copy back your edited code into the directory.

## Timeline

The exercise is usually due on Tuesday afternoon, in the week following its released.  You must submit each exercise before the deadline to receive feedback from your tutors.

The tutors have the right to refuse to read and give feedback on late submissions.

## General Advice

You are advised to (i) spend some time thinking before you begin coding, (ii) practice incremental coding, and (iii) test your programs thoroughly.

Remember to spend some time thinking about the design and approach to solving each question.

Incremental coding means do NOT type in the whole long program in a single session and then compile it. Instead, type your program in bits and pieces and compile it incrementally. Try to maintain a compilable program even while you are working on it. Submitting a compilable program that partially works is better than submitting an un-compilable one; this is especially important for your practical exams.

You should test your program thoroughly with your test data before submission.

You may assume that all input data are correct unless otherwise stated. Hence, you do NOT need to do input data validation. This is to allow you to focus on getting the program right, instead of worrying about making your program fool-proof.

## Seeking Help from Others

We encourage students to discuss and seek help from each other, from the lab tutors, or from an AI, if they are stuck.  However, do note that students are responsibile to ensure that they understand the resulting code they produce and the concepts behind them, and not just copying the code blindly.  

The exercises are meant to help you learn and practice the concepts taught in the course.  The exercises do not contribute to the final grade and therefore there is no point in plagiarizing code to solve the questions.  

All exercises are designed to be completed within half a day.  If you get stuck on an issue for longer than that, you should talk to others.

## Method of Submission

Please follow the instructions above to submit your code to the tutors for comments.  Programs submitted through other means, such as emails, will NOT be accepted.

## Use of Ed

If you have doubts about the problem statements of an exercise, you may raise them on Ed.  But before that, please read through the problem statements carefully first, and check if the same questions have been asked and answered on the forum.

Please exercise discretion when posting to Ed.  Before the deadline, if you need to post your solution, complete or partial, please use the spoiler" feature.
