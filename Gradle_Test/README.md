###Gradle单元测试

我们可以通过在Gradle添加Java插件来执行单元测试的任务，默认的，在项目中所有的测试都会被执行，如果我们只想测试其中一个类，我们可以使用Java系统属性`test.single`作为测试的名字，事实上，这个系统属性的模式是`taskName.single`，其中`taskName`是我们工程中单元测试类型的名称。以下将会看到我们如何构建单元测试

1.创建Gradle工程，并在build.gradle配置文件中加入：
```Java

	// File: build.gradle
	apply plugin: 'java'
	repositories {
	    mavenCentral()
	}
	dependencies {
	    testCompile 'junit:junit:[4,)'
	}	
	test {
	    testLogging {
	        // Show that tests are run in the command-line output
	        events 'started', 'passed'
	    }
	}
```
2.第二步，我们创建一个测试类，每个测试类一个测试方法，这样子让我们可以在后面单独的调用他们

```Java

	// File: src/test/java/com/mrhaki/gradle/SampleTest.java
	package com.mrhaki.gradle;
	
	import static org.junit.Assert.*;
	import org.junit.*;
	
	public class SampleTest {
	
	    @Test public void sample() {
	        assertEquals("Gradle is gr8", "Gradle is gr8");
	    }
	    
	}
	
	// File: src/test/java/com/mrhaki/gradle/AnotherSampleTest.java
	package com.mrhaki.gradle;
	
	import static org.junit.Assert.*;
	import org.junit.*;
	
	public class AnotherSampleTest {
	
	    @Test public void anotherSample() {
	        assertEquals("Gradle is great", "Gradle is great");
	    }
	}

```

3.为了只执行SampleTest类中的测试方法，我们必须从命令行中以Java系统属性`-Dtest.single=Sample`来执行单元测试
	
	$ gradle -Dtest.single=Sample test
	:compileJava UP-TO-DATE
	:processResources UP-TO-DATE
	:classes UP-TO-DATE
	:compileTestJava
	:processTestResources UP-TO-DATE
	:testClasses
	:test
	
	com.mrhaki.gradle.SampleTest > sample STARTED
	
	com.mrhaki.gradle.SampleTest > sample PASSED
	
	BUILD SUCCESSFUL
	
	Total time: 11.404 secs

