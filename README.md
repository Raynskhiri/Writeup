# SecuriNets ISI Friendly CTF 2023-2024

This writeup will guide you to solve the tasks:

- [Server Side Template Injection](#server-side-template-injection)
- [Local file inclusion 0 (LFI0)](#local-file-inclusion-0-lfi0)
- [Local file inclusion 1 (LFI1)](#local-file-inclusion-1-lfi1)

## Server Side Template Injection

### Confirmation of Jinja2 Usage

The title of this challenge, "Server Side Template Injection," coupled with the description, strongly suggests the use of the Jinja2 template engine in the web application. The mention of a Python web framework aligns with the fact that Jinja2 is commonly employed in Python web development.

### Supporting Argument

1. **Title Significance:** The inclusion of "Template Injection" in the title implies a vulnerability related to the rendering of templates on the server side. Jinja2 is a popular template engine in Python, making it a plausible choice for the web app's backend.

2. **Python Web Framework Hint:** The description hints at the web app being built with a Python framework. Considering Jinja2 is commonly used with frameworks like Flask and Django, it strengthens the case for Jinja2 being the template engine in play.

### Basic Jinja2 Injection Examples

In your research, you've uncovered some basic examples of Jinja2 SSTI, including:

- `{{4*4}}`: Evaluates to 16

#### Exploit Test

1. **Test Payload:** Jinja2 - Basic Injection Test.

<img src="img/1.png" alt="Payload 1" width="600" height="400">

   *Description: `{{7*7}}`*

2. **Test Result:** The application responds with the output of the '{{7*7}}' command.

<img src="img/2.png" alt="Payload 1" width="600" height="400">

### SSTI Exploit for Remote Code Execution (RCE)

After confirming the presence of SSTI in the web application, the next step is to exploit it for Remote Code Execution (RCE). One common payload for achieving RCE in Jinja2 is as follows:

```jinja
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

1. **Test Payload:** Exploit the SSTI by calling os.popen().read().

<img src="img/3.png" alt="Payload 1" width="600" height="400">

3. **Test Result:** The application responds with the output of the 'id' command.

<img src="img/4.png" alt="Payload 1" width="600" height="400">

   *Description: Confirm that the application responds with the output of the 'id' command, indicating successful SSTI execution.*

4. **Directory Listing**
   - **Payload:** Modify the payload to list the contents of the current directory using `ls`.

<img src="img/5.png" alt="Payload 1" width="600" height="400">

- **Test Result:** Execute the payload and observe the result to see the files and directories in the current location.

<img src="img/6.png" alt="Payload 1" width="600" height="400">

5. **Read Flag Contents**
   - **Payload:** Modify the payload to cat the contents of `flag.txt`.

     ```jinja
     {{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read() }}
     ```

<img src="img/7.png" alt="Payload 1" width="600" height="400">

- **Test Result:** Execute the payload and observe the result to read the contents of the `flag.txt` file: `Securinets{55t1_D0N3}`.

<img src="img/8.png" alt="Payload 1" width="600" height="400">

## Local File Inclusion (LFI0)

### Confirmation of Local File Inclusion

Confirmation of Local File Inclusion (LFI) is a crucial step in identifying and fixing vulnerabilities in web applications that use PHP. In LFI, attackers exploit improper input validation to include unauthorized files. To confirm LFI, they might use traversal sequences like "../" or null bytes (%00) to navigate directories or terminate file paths. This helps them gather information about the server environment and potentially execute malicious code. Prevention involves robust input validation, file path whitelisting, and regular software updates. Addressing LFI promptly is essential for maintaining web application security.

### Supporting Argument

> Consider a PHP script that includes a file based on user input. If proper sanitization is not in place, an attacker could manipulate the page parameter to include local or remote files, leading to unauthorized access or code execution.

1. **Identify Vulnerable Script:**

```
<?php
$file = $_GET['page'];
include($file);
?>
```

2. **Exploit Directory Traversal:**

In the following examples we include the **/etc/passwd** file, check the Directory & Path Traversal chapter for more interesting files.

```plaintext
http://example.com/index.php?page=../../../etc/passwd
```

3. **include 'flag.txt' File:**

Once I successfully traverse directories, I can include the flag.txt file.

```plaintext
http://example.com/index.php?page=../../../../flag.txt
```
4. **Explanation :**

- **../**: Move up one directory level.
- **page=../../../flag.txt**: Traverse up three levels to the root directory and include the flag.txt file.

5. **Result :**

I've now successfully included the flag.txt file, allowing me to view its content or execute any malicious code it may contain.
