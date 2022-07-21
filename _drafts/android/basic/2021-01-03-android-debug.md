
## Gradle
### Error: Cannot fit requested classes in a single dex file (# methods: 149346 > 65536)

```groovy
apply plugin: 'com.android.application'

android {
    defaultConfig {
    	//...
    	multiDexEnable true
    	//...
	}

	dependencies {
    	//...
    	implementation 'com.android.support:multidex:1.0.3'
    	//...
	}
}
```

```java
public class YourApplication extends Application {
    @Override
	public void onCreate() {
    	super.onCreaet();
    	MultiDex.install(this);
	}
}

```

### Error: More than one file was found with OS independent path 'META-INF/proguard/xxx.pro'
```groovy
apply plugin: 'com.android.application'

android {
    defaultConfig {
    	packagingOptions {
            exclude 'META-INF/proguard/xxx.pro'
    	}
	}
}
```

