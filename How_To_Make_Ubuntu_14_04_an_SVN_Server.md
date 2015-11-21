# How to Set Up a Ubuntu 14.04 SVN Server

This brief tutorial shows you how to make Ubuntu 14.04 into an SVN server.

In the tutorials/examples given by Ubuntu and others online, I found that
several steps were not included that would probably result in a headache for
new users.

This tutorial assumes you are SSH'd into your server with sudo/root
privileges.

## Steps:
1. Install updates: sudo apt-get update

2. Get the required packages: sudo apt-get install subversion apache2 libapache2-svn apache2-utils

3. Make the repository directory: sudo mkdir -p /home/svn
   The above command will be the directory where all your repositories will
go.

4. Create a test repo: sudo svnadmin create /home/svn/test

5. Change ownership for the repo: sudo chown -R www-data:www-data
/home/svn/test

6. Add the following between the <VirtualHost> tags in /etc/apache2/sites-available/000-default.conf:
   <Location />
      DAV svn
      SVNParentPath /home/svn/
      SVNListParentPath On
      AuthType Basic
      AuthName "Home Repository"
      AuthUserFile /home/svn/.htpasswd
      Require valid-user
      LimitXMLRequestBody 0
   </Location>

The above lines don't need to be added to 000-default.conf, but should be
added to a .conf file in the /etc/apache2/sites-available/ directory.

Below is an explanation of the above lines:
<Location />: This signifies that you want the root of your site to show the
SVN repos (e.g. http://www.servername.com/). You can change this to any path.
For example <Location /svn> will make the SVN repos available from
http://www.servername.com/svn.

DAV svn: Sets the authentication protocol to SVN

SVNParentPath /home/svn/: This is the path to wherever your SVN repos are
stored on the server

SVNListParentPath On: Allows you to access your repos from the common parent
path rather than the repos absolute path.

AuthType Basic: Prompts you for username/password when accessing the URL (not
100% sure about this)

AuthUserFile /home/svn/.htpasswd: Specifies the file where all the user
credentials for the site will be stored. This can be a path to any password
file and we will be creating the password later.

Require valid-user: Pretty self-explanatory. You need to be a valid user (as
defined in the password file) to access the data.

LimitXMLRequestBody 0: Avoids "413 Request Entity Too Large" response when a
client tries a large svn update.

7. Enable the site: sudo a2ensite 000-default.conf
   If you put the XML from Step 6 in another file, do: sudo a2ensite
otherFile.conf

8. Enable the SVN module: sudo a2enmod dav_svn
   This enables the SVN module for use in the Apache server. Without this,
you'll probably get the following error: Unknown DAV provider: svn

9. Create the passwords file: sudo htpasswd -cm /home/svn/.htpasswd user1
   In the above command user1 is the username of the first user you wish to
add. The command will then prompt you to enter that user's password.
   To add more users, run: sudo htpasswd -m /home/svn/.htpasswd user2

10. Restart the Apache server: sudo /etc/init.d/apache2 restart
   I recommend restarting it rather than reloading as reloading doesn't take
into effect the configuration changes you made.
