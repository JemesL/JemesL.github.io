---
title: Xcode 添加 script 自动更新 Build
date: 2018-11-27 17:47:03
tags:
---

每次打包经常会遇到忘记修改 bulid 导致上传失败，又不得不重新打包上传，极其浪费时间。
xcode 的 Build Phases 的run script 可以添加脚本，来自动更新build的值

代码如下：
```Bash
git=`sh /etc/profile; which git`
appBuild=`"$git" rev-list --all |wc -l`

if [ $CONFIGURATION = "Debug" ]; then
branchName=`"$git" rev-parse --abbrev-ref HEAD`
branchShort=`"$git" rev-parse --abbrev-short HEAD`
/usr/libexec/PlistBuddy -c "Set :BranchName $branchName" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
/usr/libexec/PlistBuddy -c "Set :BranchShort $branchShort" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
/usr/libexec/PlistBuddy -c "Set :APPBuildNum $appBuild" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
else
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $appBuild" "${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
fi
echo "Updated ${TARGET_BUILD_DIR}/${INFOPLIST_PATH}"
```

其中  **BranchName**、**BranchShort**、**APPBuildNum**、**CFBundleVersion**，前三个需要在info.plist 新增字段
也可以根据需求自定义其他字段。
这里根据 debug 和 release 做了区分。
DEBUG： 可以拿到 分支名字、 分支commitID、build次数
RELEASE：可以拿到build 次数

ps：目前，在多target版本还有点问题，在最初的target里没有问题，但是在copy出来的其他target暂时无法运行