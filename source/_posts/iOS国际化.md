---

title: iOS国际化
urlPath: iOS-localization
date: 2019-06-06
updated: 2019-06-06
tag: iOS

---

### AppName Localization

* project -> localizations add
* add file -> `InfoPlist.string` -> right action -> localization
* `"CFBundleDisplayName" = "language"`

<!-- more -->

### String Localization
* add file -> `Localizable.string` -> right action -> localization
* `"字符名称" = "stringName"`

### 正则替换&批量生成
* swift
  
  * replace -> `("[^\x00-\xff]+")`
  * with -> `NSLocalizedString($1, comment: "")`
  * terminal cd项目根目录
  * `find ./ -name '*.swift' -print0 | xargs -0 genstrings -o projectName/language.lproj/`

* oc
  
  * replace -> `(@"[^\x00-\xff]+")`
  * with -> `NSLocalizedString($1, comment: @"")`
  * terminal cd项目根目录
  * `find . \( -name '*.m' -o -name '*.h' \) -print0 | xargs -0 genstrings -o projectName/language.lproj/`
  * `find ./ -name "*.m" -print0 | xargs -0 genstrings -o projectName/language.lproj/`
 
