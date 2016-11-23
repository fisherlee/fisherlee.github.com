---
layout: post
title:  "部署app的持续集成环境"
date:   2016-06-12 12:00:00 +0800
categories: 持续集成
---

### 1 介绍
	
	使用jenkins部署app的持续集成环境。

### 2 Jenkins部署
#### 2.1 安装jenkins

- 步骤1、安装homebrew

命令：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

homebrew环境检查： 'brew doctor'

> 如果需要修改ruby镜像，参考此网址：https://ruby.taobao.org/

- 步骤2、安装jenkins

	使用homebrew安装：`brew install Jenkins`

- 步骤3、启动jenkins

	`java -jar /usr/local/opt/jenkins/libexec/jenkins.war`

	访问jenkins：`http://ip:8080`
	
> Jenkins自动启动命令
- 1）创建启动项
`ln -sfv /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents`
- 2）编辑`~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist`，修改`--httpListenAddress=127.0.0.1`为本机的局域网ip
`open ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist`

#### 2.2 jenkins基本配置

*系统管理->插件管理*，根据需要安装插件

- Email Extension Plugin

- SICCI for Xcode Plugin

- Xcode integration

- Workspace Cleanup Plugin

- fir-plugin

- Subversion Plug-in

#### 2.3 jenkins邮件配置

##### 2.2.1 基本配置

*系统管理->系统设置*，配置 **Extended E-mail Notification**，参考下图：
![pic1]()

##### 2.2.2 参数属性

- **Default Content Type**：构建后发送邮件内容的类型，有Text和HTML两种

- **Use List-ID Email Header**：为所有的邮件设置一个List-ID的邮件信头，这样你就可以在邮件客户端使用过滤。它也能阻止邮件发件人大部分的自动回复(诸如离开办公室、休假等等)。
> 你可以使用你习惯的任何名称或者ID号，但是他们必须符合如下其中一种格式(真实的ID必须要包含在<和>标记里)：
`<ci-notifications.company.org>；Build Notifications <ci-notifications.company.org>；“Build Notifications” <ci-notifications.company.org>`

- **Add 'Precedence: bulk' Email Header**：设置优先级

- **Default Recipients**：自定义默认电子邮件收件人列表。如果没有被项目配置覆盖,该插件会使用这个列表。您可以在项目配置使用*$ DEFAULT_RECIPIENTS*参数包括此默认列表，以及添加新的地址在项目级别。添加抄送：cc:电子邮件地址例如,CC:someone@somewhere.com

- **Reply To List**：回复列表

- **Emergency reroute**：如果这个字段不为空，所有的电子邮件将被单独发送到该地址（或地址列表）

- **Excluded Committers**：防止邮件被邮件系统认为是垃圾邮件,邮件列表应该没有扩展的账户名(如:@domain.com),并且使用逗号分隔

- **Default Subject**：自定义邮件通知的默认主题名称。该选项能在邮件的主题字段中替换一些参数，这样你就可以在构建中包含指定的输出信息。

- **Maximum Attachment Size**：邮件最大附件大小

- **Default Content**：自定义邮件通知的默认内容主体。该选项能在邮件的内容中替换一些参数，这样你就可以在构建中包含指定的输出信息。

- **Default Pre-send Script**：默认发送前执行的脚本

##### 2.2.3 常用属性

在**Default Content**中用到的常用属性

- **${FILE,path="PATH"}**，包括指定文件（路径）的含量相对于工作空间根目录。path文件路径，注意：是工作区目录的相对路径

- **${BUILD_NUMBER}** 显示当前构建的编号

- **${JOB_DESCRIPTION}**，显示项目描述

- **${SVN_REVISION}**，显示svn版本号。还支持Subversion插件出口的SVN_REVISION_n版本

- **${CAUSE}**，显示谁、通过什么渠道触发这次构建

- **${CHANGES }**，-显示上一次构建之后的变化

- **showPaths**，如果为 true,显示提交修改后的地址。默认false

- **showDependencies**，如果为true，显示项目构建依赖。默认为false

- **format**，遍历提交信息，一个包含%X的字符串，其中%a表示作者，%d表示日期，%m表示消息，%p表示路径，%r表示版本。注意，并不是所有的版本系统都支持%d和%r。如果指定showPaths将被忽略。默认“[%a] %m\\n”

- **pathFormat**，一个包含“%p”的字符串，用来标示怎么打印路径

- **${BUILD_ID}显示当前构建生成的ID

- **${PROJECT_NAME}**，显示项目的全名

- **${PROJECT_DISPLAY_NAME}**，显示项目的显示名称

- **${SCRIPT}**，从一个脚本生成自定义消息内容。自定义脚本应该放在"$JENKINS_HOME/email-templates"。当使用自定义脚本时会默认搜索$JENKINS_HOME/email-templatesdirectory目录。其他的目录将不会被搜索

- **script**，当其使用的时候，仅仅只有最后一个值会被脚本使用（不能同时使用script和template）

- **template**，常规的simpletemplateengine格式模板

- **${JENKINS_URL}**，显示Jenkins服务器的url地址（你可以再系统配置页更改）

- **${BUILD_LOG_MULTILINE_REGEX}**，按正则表达式匹配并显示构建日志

- **regex**，java.util.regex.Pattern 生成正则表达式匹配的构建日志。无默认值，可为空

- **maxMatches**，匹配的最大数量。如果为0，将匹配所有。默认为0

- **showTruncatedLines**，如果为true，包含[...truncated ### lines...]行。默认为true

- **substText**，如果非空，就把这部分文字（而不是整行）插入该邮件。默认为空

- **escapeHtml**，如果为true，格式化HTML。默认为false

- **matchedSegmentHtmlStyle**， 如果非空，输出HTML。匹配的行数将变为<b style=”your-style-value”> html escaped matched line </b>格式。默认为空

- **${BUILD_LOG}**，显示最终构建日志

- **maxLines**，日志最多显示的行数，默认250行

- **escapeHtml**，如果为true，格式化HTML。默认false

- **${PROJECT_URL}**，显示项目的URL地址

- **${BUILD_STATUS}**，-显示当前构建的状态(失败、成功等等)

- **${BUILD_URL}**，-显示当前构建的URL地址

- **${CHANGES_SINCE_LAST_SUCCESS}**，-显示上一次成功构建之后的变化

- **reverse**，在顶部标示新近的构建。默认false

- **format**，遍历构建信息，一个包含%X的字符串，其中%c为所有的改变，%n为构建编号。默认”Changes for Build #%n\n%c\n”

- **showPaths、changesFormat、pathFormat**，分别定义如${CHANGES}的showPaths、format和pathFormat参数

- **${CHANGES_SINCE_LAST_UNSTABLE}**，-显示显示上一次不稳固或者成功的构建之后的变化

- **reverse**，在顶部标示新近的构建。默认false

- **format**，遍历构建信息，一个包含%X的字符串，其中%c为所有的改变，%n为构建编号。默认”Changes for Build #%n\n%c\n”

- **showPaths**，changesFormat,pathFormat分别定义如${CHANGES}的showPaths、format和pathFormat参数

- **${ENV}**，–显示一个环境变量

- **var**，显示该环境变量的名称。如果为空，显示所有，默认为空

- **${FAILED_TESTS}**，-如果有失败的测试，显示这些失败的单元测试信息

- **${JENKINS_URL}**，-显示Jenkins服务器的地址。(你能在“系统配置”页改变它)

- **${PROJECT_URL}**，-显示项目的URL

- **${SVN_REVISION}**，-显示SVN的版本号

- **${JELLY_SCRIPT}**，-从一个Jelly脚本模板中自定义消息内容。有两种模板可供配置：HTML和TEXT。你可以在$JENKINS_HOME/email-templates下自定义替换它。当使用自动义模板时，”template”参数的名称不包含“.jelly”

- **template**，模板名称，默认”html”

- **${TEST_COUNTS}**，-显示测试的数量

- **var**，默认“total”

- **total**，所有测试的数量

- **fail**，失败测试的数量

- **skip**，跳过测试的数量

### 3 iOS持续集成
#### 3.1 新建一个项目

jenkins首页点击‘新建’，输入项目名称

![pic2]()

![pic3]()

#### 3.2 项目配置

##### 3.2.1源码管理

选择Subversion，在’Credentials’中添加svn的账号密码，此时的svn地址用svn://… 代替http://…。’Repository URL’中的svn地址正常填写http://...。

![pic4]()

##### 3.2.2构建触发器

Poll SCM：定时检查源码变更（根据SCM软件的版本号），如果有更新就checkout最新的，然后执行构建动作。
 
Build periodically：周期进行项目构建（它不检查源码是否发生变化）

周一到周五上午10点-11点执行构建动作

`H H(10-11)**1-5`

![pic5]()

##### 3.2.3构建环境

选择Restore OS X keychains after build process as defined in global configuration

##### 3.2.4构建

构建步骤选择Xcode

配置如下

- **Target**：工程的Target名称

- **Clean before build？**：构建前是否clean

- **Configuration**：填写Release或者是Debug

- **.ipa filename pattern**：Pay-DEV-$(VERSION)- $(BUILD_DATE) ，$(VERSION) app的版本号，$(BUILD_DATE) 构建日期

- **Output directory**：ipa的输出路径，可从改路径直接下载或者是上传ftp

- **Code Signing Identity**：证书，Configuration
的配置Debug和Release后证书不同，注意证书大小写和空格要完全一致，如果构建时候提示证书错误，则需要在钥匙串中把证书复制到“系统”下面

- **SDK**：可通过在终端通过·xcodebuild –version·命令查看

- **Build output directory**：构建后的目录，注意Configuration 配置的Debug和Release，此处填写路径的不同，$(WORKSPACE)build/Debug-iphoneos，$(WORKSPACE)build/Release-iphoneos

![pic6]()

![pic7]()

##### 3.2.5构建后操作

构建后自动发送邮件，操作选择`Extended E-mail Notification`

默认调用系统设置中已设置的好的邮件联系人，邮件内容

**Project Recipient List**：默认收件人地址，多个收件人用”,”分割，如“$DEFAULT_RECIPIENTS, test@126.com”，也可以去掉$DEFAULT_RECIPIENTS

![pic8]()

#### 3.3 iOS工程配置
##### 3.3.1 Xcode配置

targets选中Build Settings，然后修改`CODE_SIGN_RESOURCE_RULES_PATH`，加入参数`“$(SDKROOT)/ResourceRules.plist”`

