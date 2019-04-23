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



## Vulnerabilities
* #### Τηλεσυνεργασία
  
  * Vulnerability 
  * Defence
  
  
* #### Ανταλλαγή αρχείων 
  * Vulnerability 
  * Defence

### Τηλεσυνεργασία 

#### Vulnerability
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
  
#### Defence
  
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
  
### Ανταλλαγή αρχείων 
  
#### Vulnerability

  [index.php(dropbox)](modules/dropbox/index.php) ```file exchange```
  
  A malicious user can upload a file in this php page.
  ![](progress-images/upload-files-1.png)
     
  If the malicious user finds the path to the uploaded file he can run whatever he wants from the inside
  of our server.
  ![](progress-images/upload-files-2.png)
  
  [index.php(dropbox)](modules/dropbox/index.php) redirects to [dropbox_submit.php](modules/dropbox/dropbox_submit.php)
  with this form: 
  ```
  <form method="post" action="dropbox_submit.php" enctype="multipart/form-data" onsubmit="return checkForm(this)">
  tCont2;
  	$tool_content .= "
      <table width='99%' class='FormData'>
      <tbody>
      <tr>
        <th class='left' width='220'>&nbsp;</th>
        <td><b>".$dropbox_lang["uploadFile"]."</b></td>
      </tr>
      <tr>
        <th class='left'>".$dropbox_lang['file']." :</th>
        <td><input type='file' name='file' size='35' />
            <input type='hidden' name='dropbox_unid' value='$dropbox_unid' />
        </td>
   ...
   ...
   ```
   
   Then it uses this code to set the name of the file:
   ```
   $dropbox_title = $dropbox_filename;
   $format = get_file_extension($dropbox_filename);
   $dropbox_filename = safe_filename($format);
   // Transform any .php file in .phps fo security
   // $dropbox_filename = php2phps ($dropbox_filename);
   ```
   
   #### Defence
   
   We can check the file's extension and if the file is php or html type then we add an "s" to the extension
   (htmls, phps) so the malicious user cannot run php or a script inside our server.
   
   ```
   $dropbox_title = $dropbox_filename;
   $format = get_file_extension($dropbox_filename);
   $dropbox_filename = safe_filename($format);
   // saves the file extension
   $file_type = pathinfo($dropbox_filename, PATHINFO_EXTENSION);
   // if file is php or html changes extension to phps or htmls
   if ($file_type == 'php' || $file_type == 'html') {
       $dropbox_filename = substr_replace($dropbox_filename, $dropbox_filename . 's', $file_type);
   }
   ```