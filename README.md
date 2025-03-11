# Exploring-And-Remediating-XXE

<h2>Description</h2>

Understanding the basic concepts related to **XML**, exploring **XML External Entity (XXE)** and its components and learning how to exploit and remediate the vulnerability.


<h2>Languages and Utilities Used</h2>

- XML
- Burpsuite


<h2>Task with in-depth breakdown</h2>

**XML** is a file format and markup language that allows users to store, share, and exchange data. It organises data in tabs and can be visualised as files in a filing cabinet.

<img src="https://i.imgur.com/YHGokdN.png" alt="XML"/>

**DTD = Document Type Definition:** defines structure for XML such as data types, elements for servers to communicate with one another: 

<img src="https://i.imgur.com/Hdyl9fP.png" alt="DTD"/>

In the above **DTD**, <!ELEMENT>  defines the elements (tags) that are allowed, like name, address, email, and phone, whereas **#PCDATA** stands for parsed people data, meaning it will consist of just plain text.

External refs are bad most of the time: an external entity references data from an external file or resource. In the following code, the entity &thmFile; in this case opens the passwd file which would load a list of the system’s accounts. Another thing it could also do (not used as example in screenshot) is refer to an external file located e.g. at "http://tryhackme.com/robots.txt", which would be loaded into the XML, if allowed by the system:

**XXE** is an attack that takes advantage of how XML parsers handle external entities. When a web application processes an XML file that contains an external entity, the parser attempts to load or execute whatever resource the entity points to, shown below:

<img src="https://i.imgur.com/vB6wdvC.png" alt="External refs"/>

Use **burpsuite** to check the vulnerability of the web app. **Burpsuite** will proxy the web application, allowing us to be a “man in the middle” and check out each web request as it goes to the web server.

Setting within **burpsuite**: allow burp’s browser to run without a sandbox.

Go to web server. In this case: http://10.10.27.59.

Turn off intercept in **burp**, and check out website and all requests within **burpsuite**, sending them to repeater.

Once you browse the **URL**, all the requests are intercepted and can be seen under the **Proxy->HTTP** history tab in burp.

When attempting to checkout you will see a forbidden page because the details are only accessible to admins. We will try to bypass this and access other people's wishes.

Now, when you visit the URL, http://10.10.27.59/product.php, and click Add to **Wishlist**, an **AJAX** call is made to **wishlist.php** with the following **XML** as input:

<img src="https://i.imgur.com/HukPFyy.png" alt="XML Input"/>

<img src="https://i.imgur.com/aSBP7ah.png" alt="XML Input"/>

We need to check if the **XML** is vulnerable to **XXE**.

Do this by pasting in this payload:

<img src="https://i.imgur.com/G5dxEYq.png" alt="Payload1"/>

First line defines a proper xml document. “Foo” is placeholder. We are trying to read /etc/hosts/ file which is a file that resolves IPs to domain names - a good file for testing remote file read.

Change payload in XML code within burp and send request to test if vulnerable:

<img src="https://i.imgur.com/Tq3tuTY.png" alt="Payload2"/>

Result is successful:

<img src="https://i.imgur.com/F41UjMq.png" alt="Result"/>

We can use this exploit to read the wishlist file at http://10.10.27.59/wishes/wish_21.txt we don’t have the correct permissions for from above. var/www/html is file root for web server which serves and stores all html.

We edit XML code and send request to exploit this succesfully:

<img src="https://i.imgur.com/bvIJAwm.png" alt="Successful payload request"/>

By change the number in the html request (**wish_21**) within the XML code we edited: we can read wishlists of others. 

You can loop request to change number within burp by sending to intruder and editing the payload settings for the number:

<img src="https://i.imgur.com/bHSQj3G.png" alt="Loop request"/>

Check **changelog** by visiting http://10.10.27.59/CHANGELOG. 
The following proactive approach helped to address the potential risks against XXE attacks:
**Disable External Entity Loading:** The primary fix is to disable external entity loading in your **XML** parser. In **PHP**, for example, you can prevent **XXE** by setting **libxml_disable_entity_loader(true)** before processing the **XML**.

Validate and Sanitise User Input: Always validate and sanitise the **XML** input received from users. This ensures that only expected data is processed, reducing the risk of malicious content being included in the request. For example, remove suspicious keywords like **/etc/host, /etc/passwd**, etc, from the request.
