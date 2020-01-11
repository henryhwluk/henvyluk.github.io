---

title: Daily Notes
urlPath: dailynotes
date: 2019-08-12
updated: 2020-01-11
tag: [Ubuntu, linux, Gnome, VNC]

---

### logs
* cd  `/opt/xxx/logs`
*  `tail -f xxx.log`
* `grep -C10 key_word xxx.log`
* `tail -f lion.log | grep key_word`

<!-- more -->

### git

* add remote git(migrate git)
  * git remote add origin(remoteName) xxx(url)
  * git fetch origin(remoteName)
  * git branch --set-upstream-to=origin(remoteName)/remote_branch your_branch
  * git push (default current branch)

* go back git commit
  * git log
  * git reset --hard commit_id
  * git push -f origin(remoteName) your_branch

* git stash
  * git stash save "save message"
  * git stash list
  * git stash show(default first stage) /git stash show stash@{1}
  * git stash pop(default first stage stash@{0})
  * git stash drop stash@{$num} 
  * git stash clear
* git merge
  * git checkout your_feature_develop_branch
  * git pull
  * git checkout your_develop_branch
  * git merge your_feature_develop_branch
  * git push origin your_develop_branch
  * your_develop_branch pr -> public_develop_branch
* git feature&bug develop method 
  * update your branch
    * git pull origin_develop_branch
    * git pull public_develop_branch
    * git pull origin_master_branch
    * git pull public_master_branch
    * git push origin origin_develop_branch
    * git push origin origin_master_branch
  * create develop feature branch
    * origin_develop_branch -> mmdd-feature-SXXXX-dev
    * origin_master_branch -> mmdd-feature-SXXXX
  * daily develop commit
    * origin mmdd-feature-SXXXX merge to -> origin mmdd-feature-SXXXX-dev
    * slove conflicts
    * git add conflicts_file
    * git commit -m "slove conflicts file"
    * git push origin mmdd-feature-SXXXX-dev
    * origin mmdd-feature-SXXXX-dev pr to -> public public_develop_branch
  * plan commit
    * origin mmdd-feature-SXXXX pr to -> public plan-mmdd-feature-SXXXX

### golang

* dep ensure -v -update xxx

### cert

* export cert.p12 & key.p12 in keychain
* cd .p12 finder path
* cert.p12 -> apns-pro-cert.pem  `openssl pkcs12 -clcerts -nokeys -out apns-pro-cert.pem -in cert.p12`
* key.p12 -> apns-pro-key.pem `openssl pkcs12 -nocerts -out apns-pro-key.pem -in key.p12`(password must more than 4byte)
*  apns-pro-key.pem -> rsa `openssl rsa -in apns-pro-key.pem -out apns-pro-key.pem`
* merge apns-pro-key.pem & apns-pro-cert.pem -> apns-pro.pem  `cat apns-pro-cert.pem apns-pro-key.pem > apns.pem`
* zip encrypt `zip -e apns.zip apns.pem `

### ffmpeg
* `ffmpeg -i big.mov -vf scale=360:-1 small.gif`

### fastlane

* `fastlane add_plugin versioning`

### awk
* cd git
* `git log --stat | awk -f awk_file_path`

```ruby
#!/bin/awk
BEGIN{authorname="null"}
{
	if($1 =="Author:"){
	   authorname=$2;
	}
	if($5=="insertions(+)"||$5=="insertions(+),"){
	   author[authorname]+=$4;
	}

}END{
	for(a in author){
	 print(a, author[a]);
	}
}
```

### framework

```bash
lipo -create 
Release-iphoneos/gl_framework.framework/gl_framework 
Release-iphonesimulator/gl_framework.framework/gl_framework 
-output Release-iphoneos/gl_framework.framework/gl_framework
```

