---
layout: page
title: "Git Reference"
permalink: /gitref/git_on_server
---

[comment]: <> (TODO: REV MARKER)

[comment]: <> (TODO: This is raw copy paste from your notes need to format and organize.)

Git on the Server - The Protocols

	The Protocols
	
		Local Protocol
		
			§ The most basic protocol supported by Git in which the remote repository is in another directory on the same host. 
			§ This is used if everyone has access to a shared directory such as an NFS mount
			§ To clone such a repository you can use either of the two commands bellow.
				□ $ git clone /srv/git/project.git
				□ $ git clone file:///srv/git/project.git
			§ If you just specify the path, Git tried to use hard links or directly copy the files it needs.  If you use the file:/// approach Git fires up the processes that it normally uses to transfer data over a network, which is generally much less efficient.
				□ Reason to sue file:// is if you want a clean copy of the repository with extraneous references left out.
			§ To add a local repository to an existing Git project with the command below.  After running this command you can push and pull using the remote name local_proj.
				□ git remote add local_proj /srv/git/project.git
			
			The Pros
				□ file based repos are simple and use existing file permissions and network access.
				□ This is also a nice option for quickly grabbing work form someone else's working repository.
			The Cons
				□ Shared access is generally more difficult to set up and reach from multiple locations than basic network access.
				□ Local repo is only fast if you have fast access to the data.
				□ Every user has full shell access to the "remote" directory, and there is nothing preventing them from changing or removing internal Git files and corrupting the repository.
			
		The HTTP Protocols
		
			§ Git post version 1.6.6 uses what is referred to as Smart HTTP protocol which negotiates data transfer similar to how it does over SSH.  The old approach is referred to as Dumb HTTP.
				
				Smart HTTP
					• Operates very similarly to the SSH or Git protocols, but runs over standard HTTPS ports and can use various HTTP authentication mechanisms
						◊ This is often easier for the user since you can use username/passwords instead of having to set up SSH keys.
					• This has become the most popular protocol for Git since it can be set up to server both anonymously like git:// and can also be pushed over with authentication and encryption like the SSH protocol without having to set up different URLs for these things.
						◊ If you push to a repo that requires authentication the server can prompt for a username and password.  Same applies for read access.
				Dumb HTTP
					• If server does not respond to the new Smart HTTP protocol Git will try to fail back to the dumb one.
					• Dumb protocol expects the bare Git repository to be served like normal files from a web server.
					• Dumb HTTP is very simple to set up.  You just put the bare files under your HTTP document root and set up a specific post-update hook and you are done.  (hooks we will cover later)
						◊ To allow read access to your repository over HTTP do something like this:
							} 
							} The post-update hook that comes with Git by default will run the appropireat command (git update-server-info) to make HTP fetching and cloning work properly.
								– This command is run when you push to this repository.
				The Pros
					• With smart HTTP protocol you can have a single URL for all types of access and have the server prompt only when authentication is needed.
					• Being able to authenticate with username/password is also a big advantage over SSH since no keys are needed.
					• You can also server your repositories read-only over HTTPS, which means you can encrypt the content transfer; or you can go so far as to make the clients use specific signed SSL certificates.
					• HTTP and HTTPS is usually allowed on corporate firewalls so no tinkering needed.
				The Cons
					• Can be a bit tricky to set up HTTS over SSH on some servers.  Other than that there is very little advantage that other protocols have over Smart HTTP for serving Git content.
					• providing credentials can be cumbersome, but you can use Keychain access on Macs or Credential Manager on Windows.  There is a section on credential management later in the book.
				
		The SSH Protocol
			§ Common set up when self hosting.
			§ To cline a repo using SSH use ssh://
				□ git clone ssh://[user@]server/project.git
				□ or…
				□ git clone [user@]server:project.git
				□ In both cases if you don't specify a user git assumes your currently logged in user.
			Pros
				□ Easy to set up and SSH is widely used with admins knowing it well.
				□ Access over SSH is secure everything is encrypted
				□ SSH makes data compact before transferring it.
			Cons
				□ Does not support anonymous access to your Git repository.  If you are using SSH people must have SSH access to your machine even for read only.
			
		The Git Protocol
			§ Git protocol is a special daemon that comes packaged with Git.
			§ It listens on a dedicated port (9418) that provides a service similar to the SSH protocol, but with absolutely no authentication.
			§ to server over Git protocol you must create a git-daemon0export-ok file. 
				□ The deamon won't serve a repo without this file in it.  This is the only security.  
				□ Pushing is not allowed by deafult but can be configured (everyone with access to the URL will have access)
			Pros
				□ Git protocol is often the fastest network protocol available.  If you are serving a lot of traffic for a public project or serving a very large project that doesn't require user authentication for read access you want to use this approach.
			Cons
				□ Lack of authentication.  
				□ Generally you would use Git protocol for the masses downloading your repo, but use SSH or HTTPs for your developers.
		
Git on the Server - Getting Git on a Server

	• In order to initially set up any Git server, you have to export an existing repository into a new bare repository -- a repository that doesn't contain a working directory.
	• you use the --bare option for this.  By convention bare repository names end with suffix .git like so…
		○ $ git clone --bare my_project, my_project.git
		○ After this command you should now have a copy of the Git directory in your my_project.git directory.

	Putting the Bare Repository on a Server
	
		○ Once you have the bare repository you can place it on a server where people will have access via SSH or a web server setup.
		○ A simple set up for small operations is to create a single git user on a host and have all your devs sent you their public keys to be added to the authorized_keys file of the git user.
			§ This does not effect the commit data in any way.
		○ You can also use LDAP auth to the host.
	
Git on the Server - Setting Up the Server

	• You can set up a bare repository using git init --bare for exampel…
		$ cd /srv/git
        $ mkdir project.git
        $ cd project.git
        $ git init --bare
    
	• If you are using SSH access you can restrict the git user account to only Git-related activities with a limited shell tool called git-shell that comes with Git.  
		○ If you set this as the git user account's login shell instead of bash or csh then the account can't have normal access to yoru server.
			§ To do this you must first add the full pathname of the git-shell command to /etc/shells if its not already there.
		○ This will block the git user from being able to SSH to the machine.

Git on the Server - Git Deamon

	• One use case for Git Deamon is to hast a repo that is used by continuous integrations servers within yoru network.  Since no authentication is requried you don't have to add permissiosn for each host and Git Daemon is very efficient.
	• Not needed here, but in the book hey cover how to start and daemonize the Git Daemon 

Git on the Server Smart HTTP

	• Setting up Smart HTTP is basically just enabling a CGI script that is provided with Git called git-http-backend on the server.
		○ This CGI will read the path and headers sent by a git fetch or git push to an HTTP URL and determine if the client can communicate over HTTP (true for any client over version 1.6.6)
		
Git on the Server - GitWeb

	• Git comes with a CGI script called GitWeb that lets you set up a basic web-based visualizer.
	• If you want to see what GitWeb would look like for your project, Git comes with a command to fire up a temporary instance if you have a lightweight web server on yoru system like lighthttpd or webrick.
		○ On Linux machines lighthttpd is often installed
	• The command is …
		○ $ git instaweb --httpd=webrick
		○ To stop the temporary server use command …
			§ $ git instaweb -httpd=webrick --stop
	• To run this on a webserver you will need to set up the CGI script on a webserver.
