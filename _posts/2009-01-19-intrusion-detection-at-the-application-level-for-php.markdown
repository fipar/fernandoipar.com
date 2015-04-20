---
layout: post
redirect_from:
  - 2009/01/19/intrusion-detection-at-the-application-level-for-php/

status: publish
published: true
title: Intrusion detection at the application level, for PHP
author:
  display_name: fernando
  login: fernando
  email: 
  url: http://fernandoipar.com
author_login: fernando
author_email: 
author_url: http://fernandoipar.com
wordpress_id: 52
wordpress_url: http://fernandoipar.com/?p=52
date: !binary |-
  MjAwOS0wMS0xOSAyMzoxOTo1NCAtMDIwMA==
date_gmt: !binary |-
  MjAwOS0wMS0yMCAwMToxOTo1NCAtMDIwMA==
categories:
- Programming
tags:
- Programming
- security
- lamp
comments: []
---
<p>Here's <a title="Intrusion Detection System for PHP" href="http://php-ids.org/" target="_self">phpids</a>, an Intrusion Detection System for PHP.</p>
<p>According to the site, it aims to counter XSS, SQL Injection, header injection, directory traversal, RFE/LFI, DoS and LDAP attacks, and unknown attack patterns,  through it's Centrifuge component.</p>
<p>Installation is simple. Just download it, copy the lib directory to a directory in your project structure, or add it to your <code>include_path</code>.</p>
<p>I actually chose a mix of both ways, so it's automatically included when I distribute my applications.</p>
<p>Here's how to use it:</p>
<p><code><br />
ini_set('include_path',ini_get('include_path').PATH_SEPARATOR.PRIVATE_ROOT.DIRECTORY_SEPARATOR.'phpids'.DIRECTORY_SEPARATOR.'lib');<br />
</code><br />
Here we're just modifying the include path in order to add phpids.<br />
PRIVATE_ROOT is a constant that I've defined in my app, which defines the root of the application. This is not accessible from the web server (following the recommendation of the <a href="http://phpsec.org/projects/guide/1.html#1.4.1">Dispatch</a> method, from the <a href="http://phpsec.org">PHP Security Consortium</a>. However, this is a particular case, in most situations, I'd recommend using a framework that already takes a similar pattern into account, like <a href="http://cakephp.org/">CakePHP</a>).<br />
phpids is where I've copied this IDS. The downloaded package has a similar root directory name, like phpids-x.y.n, depending on the version number.</p>
<p><code><br />
require_once 'IDS/Init.php';<br />
</code></p>
<p>Require the Init.php file</p>
<p>Then, whenever you need to access the request variables (in my case, this was just in one point in my application so I just had to modify one function), add something like this:</p>
<p><code><br />
$request = array(<br />
'REQUEST' =&gt; $_REQUEST,<br />
'GET' =&gt; $_GET,<br />
'POST' =&gt; $_POST,<br />
'COOKIE' =&gt; $_COOKIE<br />
);<br />
$init = IDS_Init::init(PRIVATE_ROOT. DIRECTORY_SEPARATOR .'phpids'.DIRECTORY_SEPARATOR.'lib/IDS/Config/Config.ini');</code></p>
<p>$ids = new IDS_Monitor($request, $init);<br />
$result = $ids-&gt;run();</p>
<p>if (!$result-&gt;isEmpty()) {<br />
trigger_error($result);<br />
}</p>
<p>If the $result object is not empty, the IDS detected an attack attempt and therefore you should stop processing the request.</p>
<p>And now the fun part.</p>
<p>In order to test this, I set up a very simple, very insecure test page.<br />
Here's the php code:</p>
<pre>error_reporting(E_ALL);
ini_set('include_path',ini_get('include_path').':'.'../src/phpids/lib');
require 'IDS/Init.php';
function checkIds()
{
   $request = array(
     'REQUEST' =&gt; $_REQUEST,
     'GET' =&gt; $_GET,
     'POST' =&gt; $_POST,
     'COOKIE' =&gt; $_COOKIE
);

$init = IDS_Init::init('/home/fipar/workspace/at-intranet/src/phpids/lib/IDS/Config/Config.ini');
$ids = new IDS_Monitor($request,$init);
$result = $ids-&gt;run();
if (!$result-&gt;isEmpty()) {
     trigger_error($result);
}
}

if (isset($_REQUEST['sql'])) {
     checkIds();
     $sql = $_REQUEST['sql'];
     print 'Form says '.$sql.'
';
}</pre>
<p>The html page has just an input type text with the name 'sql', and a submit button.</p>
<p>I tried the following inputs:</p>
<ul>
<li>Hello, which goes by ok</li>
<li>' and 1, which generates errors for the REQUEST and POST variables, stating<br />
<blockquote><p>Detects classic SQL injection probings 1/2 | Tags: sqli, id, lfi | ID: 42</p></blockquote>
</li>
<li>&lt;a href="http://www.google.com"&gt;www.google.com&lt;/a&gt;, which generates errors for the REQUEST and POST variables, stating<br />
<blockquote>
<ul>
<li>finds html breaking injections including whitespace attacks | Tags: xss, csrf | ID: 1</li>
<li>Detects JavaScript object properties and methods | Tags: xss, csrf, id, rfe | ID: 17</li>
<li>Detects basic SQL authentication bypass attempts 2/3 | Tags: sqli, id, lfi | ID: 45</li>
</ul>
</blockquote>
</li>
</ul>
