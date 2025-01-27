---
title: Reading Emails from a Mailbox with python
date: "2020-02-14T22:12:03.284Z"
description: "Tutorials on how to read emails with  Python"
layout: post
comments: true
categories: tech

---

### The Problem

{% include image.html name="cover-image.jpg" caption="email" %}

Recently I received a request from my client. 
He requested that I  write a script that connects to a specific mailbox, check if there are new emails, read them, and download the attachments and perform some data analysis on those attachments.
I was super excited about this task, and I asked myself; is it possible to read emails using Python? 

### The solution
I started googling, and I was lucky to find that it is possible to read emails using Python.

In this blog post, I will explain how I managed to read emails from an IMAP mail server, and how I downloaded the attachments and saved them into a  specific folder.

If you are familiar with Python, this tutorial is for you. To complete this tutorial, you need to have python 3.6 installed in your laptop and virtual environment installed.


#### First things first; create the project and initialize your virtual environment

Let us create a project we will be using :
Navigate to your command line and type the following lines of code :
`mkdir read_emails_project & cd read_emails_project`

Once in the project, let’s initialize git :

`git init`

Once we initialized git, let’s create a virtual environment:
`virtualenv -p python .venv`

You can activate the virtual environment with the following line of code :

`source .venv/bin/activate`

It's not a must to use a virtual environment. You can use your preferred environment manager; I just prefer to use virtual environments.

Once you complete the setup, we can start with the exciting part of the project.
#### Never Write secrete credentials into code, use environment variables 

Since we are reading emails from an inbox, we will need email credentials. From where are we reading those credentials?
One thing for sure; we are not putting them into the code, this is a BAD PRACTISE !!! 
The developer community can punish you for writing credentials or secret code directly into a codebase.
We will be using `.env` file, whose content will be known to us only, and we will never share it with anyone online, even putting it in a private git repository.

Let us create the `.env` file 

`touch .env`

And create a gitignore file to tell git which files to ignore.

`touch .gitignore`

open the .gitignore and add the following lines  it to ignore the `.env` file :
```
    .venv
    env
```
In those lines, we are instructing git to ignore our `.env` files and our virtual environments.

Now we can populate the `.env` file with our credentials :

```
USER_EMAIL='your@email.com'
USER_PASSWORD='your secret password.'
```
We have recorded  our email credentials in the `.env` file; let us now write a method that reads them from there.
For this task, we will be using a Python package called [python-dotenv](https://github.com/theskumar/python-dotenv)

let’s install it with :
`pip install -U python-dotenv`

let us write a function that reads and returns those environment variables : 

You can create a file with a name `utils.py` and add the following lines of code : 

```python
import os
from dotenv import load_dotenv

def read_credentails():
    """
    Return user’s credentials from the environment variables file and 
    raise a an exception if the credentials are not present 
    
    Raises:
        NotImplementedError: [description]
    """
    load_dotenv()

    USER_EMAIL = os.getenv("USER_EMAIL")
    USER_PASSWORD = os.getenv("USER_PASSWORD")
    if USER_EMAIL and USER_PASSWORD:
        return USER_EMAIL, USER_PASSWORD
    else:
        raise ValueError('Please add a .env file and write the credentials it it ,
                         refer to the sample')
```

From that function, you can notice that we are using the `load_env` function, it loads the environment variables from our `.env` file and exposes them to system environment variables so we can be able to read them from there.
After reading the environment variables, the function returns them as a tuple.
In the next section, we are going to perform the most exciting part of this tutorial; it involves reading emails and downloading attachments.

#### Reading emails:

I was happy to find that Python has a built-in feature that enables us to connect to a mailbox. It is in the `imaplib` module.
It also has a module to parse emails.

Let's create a file `reading_emails_scripts.py` inside it; we are about to perform all the magic we need.

Let us import the modules we need :

```python
from email import message_from_bytes
from imaplib import IMAP4_SSL
from .utils import read_credentails
```

We are importing the : 
- message_from_bytes function from the email module. It will help us to read the emails which come as bytes and convert them to text.
- IMAP4_SSL: class which is the main class that will help us to perform all the operations.
Note that we are using IMAP4 protocol to read emails, IMAP4 is a mail protocol used to access a mailbox on a remote server from a local email client. IMAP can be more complex but provides more conveniences for syncing across multiple devices.
You can read more about emails protocol [here](https://www.navigator.ca/support/imap-pop3-smtp/) 
- our read_credentials function, which we introduced in the previous section.

I have created a function that perform the operation and return a generator with all mails found in the mailbox , here it is :

```python
def get_unseen_emails(email_address, password):
    """
    Filter the email and return unseen emails
    Args:
        email_address (string): recipient email
        password (password): recipient password
    """
    with IMAP4_SSL("your_imap_server") as mail_connection:
        mail_connection.login(email_address, password)
        mail_connection.list()
        mail_connection.select('INBOX')
        (retcode, messages) = mail_connection.search(None,
            '(OR (UNSEEN) (FROM your.email@gmail.com))')
        if retcode == 'OK' and messages[0]:
            for index, num in enumerate(messages[0].split()):
                typ, data = mail_connection.fetch(num, '(RFC822)')
                message = message_from_bytes(data[0][1])
                typ, data = mail_connection.store(num, '+FLAGS', '\\Seen')
                yield message
```
You can see that we are instantiating a secure mail_connection from the IMAP4SSL class and using it with the python context manager.
We will be using the mail_connection object to perform all the operations we want to our mailbox.

After instantiating, we log in to the mailbox with our credentials, then we list all the mailbox categories we have, e.g.: In Google, we have INBOX, SPAM, UPDATES, FORUMS, SPAMS, etc
We select only the inbox folder, and from the inbox, we filter only unseen messages or emails coming from your email address.
Note that you can search for anything in your inbox and even complicated queries.

The search returns retcodes with messages and their IDS 
Note that : 
>The server assigned messages id to emails, and are implementation-dependent. The IMAP4 >protocol makes a distinction between sequential ids for messages at a given point in time >during a transaction and UID identifiers for messages, but not all servers seem to bother.

Once we have messages ids, we can fetch ids and the format of the headers we want.
We are using RFC822 to fetch the entire message as an RFC 2822 formatted message.
RFC 2822 is a protocol for standard messages sent between computers; you can read more about it [here](https://tools.ietf.org/html/rfc2822.html)

In the next line, we are using the message_from_bytes function from the email module to convert the bytes we received as a message.

Then we mark the message as seen by using the store method
and we yield the results.
This means the function will return an iterator which is an object we can iterate over. Find more about iterators and generators in python [here]([https://stackoverflow.com/questions/9884132/what-exactly-are-iterator-iterable-and-iteration](https://stackoverflow.com/questions/9884132/what-exactly-are-iterator-iterable-and-iteration)) and [here]([https://www.programiz.com/python-programming/generator](https://www.programiz.com/python-programming/generator)).

Once we have retrieved the emails, let us create the function that gets attachments from that email.

### Getting attachment from the email:

```python
def get_mail_attachments(message, condition_check):
    """
    Get attachments files from mail
    Args:
        message (email ): email object to retrieve attachment from
        condition_check ([function]): the function to use when filtering the
        email should return specific condition we are filtering
    Returns:
        [filename, file]: the file name, input stream from the files
    """
    for part in message.walk():
        if part.get_content_maintype() == 'multipart':
            continue
        if not part.get('Content-Disposition'):
            continue
        file_name = part.get_filename()
        if condition_check(file_name):
            yield part.get_filename(), part.get_payload(decode=1)
```
This function takes the email object yielded by the  last function and a filter function which tells which extension we can filter from the email, iterate over all his part using the walk() method. For each part we check if the main type is not multipart , and content disposition is  None

If none of those conditions is satisfied, we get the file name and return the byte stream from the email.

#### Putting everything together:
We now have all the building blocks of the application. Let’s put them in the main function:

create a file called `run.py` and add the following code inside: 

```python
from utils import read_credentails
from reading_emails_scripts import get_mail_attachments, get_unseen_emails
if __name__ == "__main__":
 email_address, password = read_credentails()
    messages = get_unseen_emails(email_address, password)
    if messages:
        for message in messages:
            attachments = get_mail_attachments(message,
                                              lambda x: x.endswith('.xml'))
            for attachment in attachments:
                if attachment:
                    with open('./data/xml_files/{}'.format(attachment[0]), 'wb') as file:
                        file.write(attachment[1])
```
We can see that; we call read_credentials which returns the credentials from the .env file,
We call the get_unseen_emails method which returns our messages , we check if the message has attachments with get_mail_attachment function and if there is an attachment we save it to a folder of our choice.

### Conclusion 

In this tutorial, we went through the whole process of reading emails with Python, we saw how to download attachments from emails, and we put everything together to have a fully working script.

It's also a good practice to write unit tests to test the code you are writing because I believe that something that is not tested is broken.

In the next part, we will see how you can write unit tests for this piece of code. So stay tuned for more.

You can check the code for this tutorial from this [Github repository]([https://github.com/espoirMur/read_emails_with_python](https://github.com/espoirMur/read_emails_with_python)).

Cheers!


#### References :
- Python Module of the week [imaplib](https://pymotw.com/2/imaplib/)
- [Read Emails and Download attachments ](https://medium.com/@sdoshi579/to-read-emails-and-download-attachments-in-python-6d7d6b60269)
- [Read only new emails in Python on StackOverflow](https://stackoverflow.com/a/29629741/4683950)
