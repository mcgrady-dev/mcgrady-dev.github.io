# aar

aar文件时在jar文件之上开发的。之所以有它是因为有些Android Library需要植入一些安卓特有的文件，比如AndroidManifest.xml，资源文件，Assets或者JNI。这些都不是jar文件的标准。

因此aar文件就时发明出来包含所有这些东西的。总的来说它和jar一样只是普通的zip文件，不过具有不同的文件结构。jar文件以classes.jar的名字被嵌入到aar文件中。其余的文件罗列如下：

\- /AndroidManifest.xml (mandatory)
\- /classes.jar (mandatory)
\- /res/ (mandatory)
\- /R.txt (mandatory)
\- /assets/ (optional)
\- /libs/*.jar (optional)
\- /jni/<abi>/*.so (optional)
\- /proguard.txt (optional)
\- /lint.jar (optional)

