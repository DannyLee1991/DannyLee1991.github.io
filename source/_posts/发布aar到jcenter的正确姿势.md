title: 发布aar到jcenter的正确姿势
tags:
  - Android
categories:
  - 工程开发
  - Android
comments: true
date: 2016-12-03 07:21:58
---

前段时间开发了一个android测试小插件[ATestKit](https://github.com/DannyLee1991/ATestKit)，准备发布到jcenter库中，可期间碰壁无数，折腾了两天最终终于上传成功。

下面是我的最终整理：

## 上传aar到jcenter的正确姿势

打开[https://bintray.com](https://bintray.com)。

### 第一坑：你需要一个vpn。

注册时国内的qq邮箱，163邮箱通通用不了，gmail可用。

天朝国情，访问google需要翻墙，不解释。

### 第二坑：注意注册入口

首页有一个**START YOUR FREE TRIAL**:

![](/img/16_12_03/001.png)

如果你按照网上搜到排名很靠前的的其他教程来进行，比如说这几篇：

- [Android Library上传到JCenter仓库实践](http://blog.csdn.net/wwj_748/article/details/51913280)
- [Android 快速发布开源项目到jcenter](http://blog.csdn.net/lmj623565791/article/details/51148825)
- [使用Gradle发布aar项目到JCenter仓库](http://www.cnblogs.com/qianxudetianxia/p/4322331.html)
- [Android Studio提交库至Bintray jCenter从入门到放弃](http://www.jianshu.com/p/31410d71eaba)

那么恭喜你，你成功的入坑了，你会发现按照他们的配置一步步进行，但到最后就是不成功，而且错误的提示也很不明确，让你找不到一点头绪。

仔细看看，你看到的jcenter的个人主页似乎和他们的不太一样。

原因可能是因为bintray.com这个网站改版了，导致注册流程和以前不一样了，如果你点击上图的那个注册入口，会引导你创建一个组织，然后你只能在组织下新建自己的仓库，而上面的几篇文章根本就没有组织这一说啊。

在这里有一个关于网站首页的说明：

[https://bintray.com/docs/usermanual/starting/starting_gettingstarted.html#_the_bintray_homepage](https://bintray.com/docs/usermanual/starting/starting_gettingstarted.html#_the_bintray_homepage)

注意看这里：

![](/img/16_12_03/003.png)

好吧，难道说明我们还有另外一个针对于open source plan的注册入口吗？这个入口听起来有点像上面几篇文章描述的那样啊。

果然有！

继续回到首页，拉到页面最底端，有另外一个入口：

![](/img/16_12_03/004.png)

从这里注册进入，你就可以不用创建组织了。

### 第三坑：收费？！

如果你不幸点击了首页的**START YOUR FREE TRIAL**，那么你的首页上会有这么一个奇怪的标识：

![](/img/16_12_03/007.png)

点进来看一看，虽然没有明确说免费版到期后会怎样，但手动终止免费版后会删掉你库里所有的东东：

![](/img/16_12_03/008.png)

如果想继续使用，那么150刀一月。

**没想到jcenter如此恶毒的把免费试用入口放在最显眼的位置，而且不明确告诉你试用账号到期后的后果，并且把免费使用的社区版入口藏的那么深。**

### 正常配置流程

通过社区版注册入口进入之后，就没有了奇怪的标识，我们可以创建一个个人仓库了：

![](/img/16_12_03/009.png)

配置选择public，类型选择Maven：

![](/img/16_12_03/010.png)

然后创建新的Package：

![](/img/16_12_03/011.png)

填写相关信息：

![](/img/16_12_03/012.png)

在你的编辑你android项目根目录下的`build.gradle`

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'
        // 添加上传到jcenter所需的插件
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.1'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

编辑library下的`build.gradle`:

```
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

version = "0.2"	//aar的版本号

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    lintOptions {
        abortOnError false
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.4.0'
}

def siteUrl = 'https://github.com/DannyLee1991/ATestKit'    // 项目主页
def gitUrl = 'https://github.com/DannyLee1991/ATestKit.git' // 项目的git地址
def module_name = 'ATestKit'	// 项目的名称
group = 'com.dannylee'	// 所在组

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                name 'ATestKit' // 名称
                url siteUrl
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0' // 开源协议名称
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt' // 协议地址
                    }
                }
                developers {
                    developer {
                        id 'dannylee'	// 账号
                        name 'dannylee'	// 名称
                        email 'leejianan1@gmail.com' // 邮箱地址
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives sourcesJar
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
	 // 读取配置文件中的用户名和key
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"		// 你在bintray上创建的库的名称
        name = module_name               // 在jcenter中的项目名称
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

回到bintray，在你的个人信息配置页中，查看APK Key：

![](/img/16_12_03/013.png)

![](/img/16_12_03/014.png)

在项目根目录下创建`local.properties`配置文件（如果有就直接打开），写入账号信息：

![](/img/16_12_03/015.png)

由于key是比较重要的信息，不能泄露出去，所以需要编辑`.gitignore`添加过滤规则来把它过滤掉：

```
# Local configuration file (sdk path, etc)
local.properties
```

然后命令行进入到你的library项目中执行：

```
gradle build
```

比较坑的是有的时候可能会失败，多试几次就会成功。

成功之后执行:

```
gradle bintrayUpload
```

如果不出意外的话，会成功上传，如果失败了，很有可能是你的网络问题，换个vpn试试说不定可以成功。如果是其他原因的失败，请自行google。

回到jcenter中项目的管理页面，就可以看到上传的版本信息了，点击**Add to JCenter**就可以正式上传到jcenter中了：

![](/img/16_12_03/016.png)

上传后，需要系统审核大概半小时，之后就可以在你的项目中使用这个aar了：

```
dependencies {
    ...
    compile 'com.dannylee:atestkit:0.2'
    ...
}
```

---

如果你选择的是第一个注册入口，流程也是一样的，唯一不同的就是在library中的`build.gradle`中要填写组织名称，否则也找不到:

```
install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
            	   userOrg 'your organisation name' // 填写组织名称
                ...
                    }
                }
                ...
            }
        }
    }
}
```




