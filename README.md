# guardians-of-apps

## Potential threats


* [index.php](index.php) ```login form, search form ```
* [newuser.php](modules/auth/newuser.php) ```new user form```
* [newprof.php](modules/auth/newprof.php) ```new profile form```
* [lostpass.php](modules/auth/lostpass.php) ```new password form```
* [profile.php](modules/profile/profile.php) ```update profile form```
* [index.php(dropbox)](modules/dropbox/index.php) ```file exchange```
* [work.php](modules/work/work.php) ```assignment submit form```
* [newtopic.php](modules/phpbb/newtopic.php) ```new subject form```
* [conference.php](modules/conference/conference.php) ```e-cooperation```






#Τηλεσυνεργασία 

##Vulnerability
  [conference.php](modules/conference/conference.php) ```e-cooperation```
  
  ![](progress-images/e-cooperation.png)
  
  
  
  This part of the application is vulnerable to XSS attacks as it doesn't have special characters escape.
  [conference.php](modules/conference/conference.php) redirects to [messageList.php](modules/conference/messageList.php)
  with this form 
  ```
  <form name='chatForm' action='messageList.php' method='get' target='messageList' onSubmit='return prepare_message();'>
    <table width='99%' class='FormData'>
    <thead>
    <tr>
      <th>&nbsp;</th>
      <td>
  
        <b>$langTypeMessage</b><br />
        <input type='text' name='msg' size='80'style='border: 1px solid #CAC3B5; background: #fbfbfb;'>
        <input type='hidden' name='chatLine'>
        <input type='submit' value=' >> '>
  
      </td>
    </tr>
    <tr>
      <th>&nbsp;</th>
      <td><iframe frameborder='0' src='messageList.php' width='100%' height='300' name='messageList' style='background: #fbfbfb; border: 1px solid #CAC3B5;'><a href='messageList.php'>Message list</a></iframe></td>
    </tr>
    </thead>
    </table>
  </form>
  ```
  
  Then it uses a writer to keep the data from the chat 
  
  ```
  // add new line
  if (isset($chatLine) and trim($chatLine) != '') {
  	$fchat = fopen($fileChatName,'a');
  	$chatLine = mathfilter($chatLine, 12, '../../courses/mathimg/');
  	fwrite($fchat,$timeNow.' - '.$nick.' : '.stripslashes($chatLine)."\n");
  	fclose($fchat);
  }
  ```
  
  In this case malicious users can XSS attack using a script.
  
##Defence
  
  We can use htmlspecialchars to prevent malicious scripts being saved as data.
  
  ```
  // add new line
  if (isset($chatLine) and trim($chatLine) != '') {
  	$fchat = fopen($fileChatName,'a');
  	$chatLine = mathfilter($chatLine, 12, '../../courses/mathimg/');
  	// Added htmlspecialchars to prevent malicious user to do xss attack on chat
  	fwrite($fchat,$timeNow.' - '.$nick.' : '.stripslashes(htmlspecialchars($chatLine, ENT_QUOTES, 'UTF-8'))."\n");
  	fclose($fchat);
  }
  ```
  
#Ανταλλαγή αρχείων 
  
##Vulnerability

  [index.php(dropbox)](modules/dropbox/index.php)
     
  ![](progress-images/upload-files-1.png)
     
  ![](progress-images/upload-files-2.png)