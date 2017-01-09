---
layout: post
title:  "iOS持续集成-jenkins自动打包并上传到ftp服务器"
date:   2017-01-09 14:50:11 +0800
categories: 持续集成
---

#### 关于"Warning: –resource-rules has been deprecated in Mac OS X >= 10.10!"

我的mac系统版本是10.12，jenkins的Xcode插件，如果选了“Pack application and build .ipa”，也就是build完成后生成ipa，总是报错"Warning: --resource-rules has been deprecated in Mac OS X >= 10.10!",搜到了github上面的解决方案：

[url：http://stackoverflow.com/questions/26459911/resource-rules-has-been-deprecated-in-mac-os-x-10-10](http://stackoverflow.com/questions/26459911/resource-rules-has-been-deprecated-in-mac-os-x-10-10)

    Remove CODE_SIGN_RESOURCE_RULES_PATH=$(SDKROOT)/ResourceRules.plist,Find the /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/PackageApplication script and update it.Find the lines including the following code in the script

`my @codesign_args = ("/usr/bin/codesign", "--force", "--preserve-metadata=identifier,entitlements,resource-rules","--sign", $opt{sign},"--resource-rules=$destApp/ResourceRules.plist");`

    change it to:


`my @codesign_args = ("/usr/bin/codesign", "--force", "--preserve-metadata=identifier,entitlements","--sign", $opt{sign});`

按照上面的方法修改，然后Build还是报错，这次错误就是“error: /usr/bin/codesign --force --preserve-metadata=identifier,entitlements --sign iPhone Distribution: XXXXXXX --entitlements ”

搜了下没找到解决办法，既然网上找不到，那就自己来写脚本吧。

先看下jenkins的Xcode工程相关的build目录，已经有了xxx.cpp，那么只要把这个xxx.cpp变成ipa就可以了，这样就简单多了，自己写个shell，在Build完成后将xxx.cpp转换成ipa文件，然后在自动上传到ftp服务器，这样就齐活了。

#### 生成ipa的shell

假设工程名字叫做"Fisher"

{% highlight shell %}
#!/bin/bash

echo "############ starting ipa ############"

jenkins_workspace_path="/Users/lee/.jenkins/workspace/Fisher/"
project_path=${jenkins_workspace_path}"Fisher/"
info_plist_path=${project_path}"Fisher"/"Info.plist"

#build short version
bundle_short_version=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" ${info_plist_path})

#build version
bundle_version=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${info_plist_path})

#build file path
build_file=${jenkins_workspace_path}"build/Release-iphoneos/Fisher.app"

#### output
#date
abc_date="`date +%Y-%m-%d:%H-%M-%S`"

#ipa name
ipa_name="Fisher-v"${bundle_version}"("${abc_date}")"".ipa"

#output path
output_path="/Users/lee/Documents/jenkins/iOS/"

#output file path
output_file=${output_path}${ipa_name}


xcrun -sdk iphoneos10.2 PackageApplication -v ${build_file} -o ${output_file}

echo "############ end ipa ############"
{% endhighlight %}

#### 上传到ftp

jenkins的自动打包到本地目录，如果再加上自动上传到ftp服务器，这才算完整。
很简单，在打包的shell后面写几句代码就可以了，这个shell可以截取ipa文件名字中的版本号，
然后创建一个目录。

{% highlight shell %}

##### ---------------- upload ftp ----------------

floder_name="v${ol_ipa_name:8:5}"
file_upload="/user/debug/${floder_name}/"

lftp -u "username","password" sftp://192.116.1.110 <<EOF
cd user/debug/

mkdir -p ${floder_name}

ls

cd ${file_upload}
put ${output_file}

quit
EOF

echo "upload ftp 192.116.1.110 done"

{% endhighlight %}

#### 执行脚本
上面的两个可以写道一个shell里面，先生成ipa，然后ftp上传。
执行这个shell也简单，在jenkins工程配置页面，Xcode配置的下面增加构建步骤，
选择“Execute shell”,然后写上`bash -x -s < /Users/lee/fisher.shell`
